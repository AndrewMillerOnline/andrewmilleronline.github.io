---
title: "Building Ionic Capacitor Apps for Release"
tags: [Development]
style: fill
color: info
description: Blunders in properly minifying with Ionic and Capacitor
---

I’ve been playing around with Ionic for a while now, and recently started using Capacitor while working on a small-scale Android medicine reminder app MedReminder. I haven’t tried Ionic’s AppFlow for managing DevOps yet, but I’ve found that Capacitor certainly makes the app signing process easier since I can just let Google Play manage the signing.

However, after publishing to the app store for the first time, I was surprised by the size of my app. It’s a simple Ionic app with only a few pages and very few additional plugins, but was pushing 11 MB. Since then, I was able to reduce the app size by 60%, down to only 4 MB, but I quickly discovered that there is a right and wrong way to do that.

## The Wrong Way to Minify

When you run `npx cap open android`, you’ll see a build.gradle file that specifies a property called minifyEnabled. Changing this value to true did two things: first, it reduced my app from 11 MB down to 10MB. Second, it broke the app completely and resulted in instant crashing across every test client. Here’s a [GitHub issue](https://github.com/ionic-team/capacitor/issues/739) with the same behavior, which is purportedly due to ProGuard.

## The Right Way to Minify

There’s a better approach to minifying that — while not immediately obvious from the Ionic and Capacitor documentation (at least, to me) — is facepalmingly obvious in hindsight. Simply use the following workflow when you’re ready to deploy your production build:

```javascript
ionic build --prod
npx cap copy
npx cap open android
```

Then in Android Studio, do Build > Generate Signed Bundle / APK and select Android App Bundle. Select your upload key, then the ‘release’ build variant and you’re good to go.  The final APK size is significantly smaller, which should help to [increase downloads and decrease uninstalls](https://medium.com/googleplaydev/shrinking-apks-growing-installs-5d3fcba23ce2).

![](/assets/app-size.png)

```
Version 1.0.1: no -prod flag, minifyEnabled=true
Version 1.1.0: no -prod flag, minifyEnabled=false
Version 1.2.0: with -prod flag, no minifyEnabled (since -prod does its own minifying)
```