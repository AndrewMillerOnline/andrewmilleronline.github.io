---
title: "Automatic Image Resizing with Google Cloud Functions"
tags: [Development]
style: border
color: info
description: Blunders in properly minifying with Ionic and Capacitor
---

I’m using Firebase as the backend for my current project, Viromob, and recently implemented functionality that will automatically resize any images uploaded to Cloud Storage. There are a few options available online for implementing this, but unfortunately both had some issues that need resolving. For posterity, here’s what I did.

## Option 1: Use Firebase’s Resize Images Extension

Google makes this incredibly easy, and you can install the extension from either the Firebase console or via the firebase CLI by running:

```
firebase ext:install storage-resize-images --project=projectId_or_alias
```

The extension takes a few minutes to install and works rather well, with one small hiccup: the newly created images aren’t publicly visible. Looking at the source code below, you can see that the download token needed to view the resized images is explicitly deleted. While I suppose there could be a security justification for this (explicit deny all then manually requesting access via the makePublic() method does seem more secure), it makes the extension unsuitable for my use case: I want auto-generated thumbnails of uploaded images that will be visible immediately. Here’s the relevant [GitHub issue](https://github.com/firebase/extensions/issues/140).

## Option 2: Create your own Cloud Function

Ultimately I ended up making my own function, with many thanks to [Jeff Delaney](https://fireship.io/lessons/image-thumbnail-resizer-cloud-function/) for providing the starting point.

```javascript
const { Storage } = require('@google-cloud/storage');
const { tmpdir } = require('os');
const { join, dirname } = require('path');
const sharp = require('sharp');
const fs = require('fs-extra');
const UUID = require("uuid-v4");
const gcs = new Storage();

/** 
* Resizes all images uploaded to Storage into widths of 160, 320, and 720 
* Inspired by https://fireship.io/lessons/image-thumbnail-resizer-cloud-function/
*/
exports.generateThumbs = functions
    .runWith({ memory: "2GB", timeoutSeconds: "60" })
    .storage
    .object()
    .onFinalize(async object => {
        const bucket = gcs.bucket(object.bucket);
        const filePath = object.name;
        const fileName = filePath.split('/').pop();
        const bucketDir = dirname(filePath);

        const workingDir = join(tmpdir(), 'thumbs');
        const tmpFilePath = join(workingDir, fileName);

        //Prevent infinite loops by checking if file has already been resized
        if (fileName.includes('_resize@') || !object.contentType.includes('image')) {
            console.log('exiting function');
            return false;
        }

        // 1. Ensure thumbnail dir exists
        await fs.ensureDir(workingDir);

        // 2. Download Source File
        await bucket.file(filePath).download({
            destination: tmpFilePath
        });

        // 3. Resize the images and define an array of upload promises
        const sizes = [160, 320, 720];

        const uploadPromises = sizes.map(async size => {
            const ext = fileName.split('.').pop();
            const oldName = fileName.replace(`.${ext}`, '');
            const newName = `${oldName}_resize@${size}.${ext}`;
            const thumbPath = join(workingDir, newName);

            // Resize source image
            await sharp(tmpFilePath)
                .resize({ width: size })
                .toFile(thumbPath);

            // Upload to GCS
            return bucket.upload(thumbPath, {
                destination: join(bucketDir, newName),
                metadata: {
                    metadata: {
                        firebaseStorageDownloadTokens: UUID()    //This makes the image public
                    }
                }
            });
        });

        // 4. Run the upload operations
        await Promise.all(uploadPromises);

        // 5. Cleanup remove the tmp/thumbs from the filesystem
        return fs.remove(workingDir);
    });
```

This function applies to any image created in your entire storage bucket (even the images created by the function), so it’s important to prevent an infinite loop. I’ll probably update my code to check against a metadata flag for resized images, rather than the name, to prevent edge cases where a user might upload an image that happens to trigger the function exit. Also, since this is bucket-wide, that function exit is a good place to check the current directory if you only want to resize certain images.

Additionally, since this function uses Sharp rather than ImageMagick which is used in the Extension, it might end up being [much faster](https://sharp.pixelplumbing.com/performance). Since I’m just tinkering around on an MVP and don’t have much in the way of users yet, I can’t confirm that though.