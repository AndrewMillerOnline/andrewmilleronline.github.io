---
name: Dataflow Extractor
title: Dataflow Extractor
tools: [HTML, Javascript, Tableau CRM]
image: /assets/tcrm-dataflow-extractor.png
description: Web tool to assist in Tableau CRM dataflow development
categories: [Projects]
hidden: true
---

## About

I created this simple web tool to assist in Tableau CRM/Einstein Analytics dataflow development.  It allows users to view the datasets being registered within a given dataflow, and then isolate only the nodes required to output a selected dataset(s).

## The Use Case

If you're in an organization like mine, you've probably seen some wicked large dataflows.  We're talking hundreds of nodes and multi-hour runtimes.  But if you're working on one particular aspect of the dataflow, say adding a few new variables to a computeExpression node, you'll have to run the entire dataflow just to test and iterate on the node which you're adding.

![tcrm dataflow extractor screenshot](/assets/tcrm-dataflow-extractor.png)

Doing things that way can equate to multi-hour turnarounds and a very slow process.  This is where the dataflow extractor shines.  Upload the JSON file for your dataflow into the extractor, and select the dataset you're modifying.  The extractor will output a new dataflow JSON containing just the nodes needed for your dataset.  Now you can work on your computeExpression in this smaller dataflow, which will have a much shorter runtime and allow you to iterate on your changes much more rapidly.  When you're done, simply copy the changes back into your original dataflow.

## The Manual Case

Of course, you can do all of this manually, too.  But manually removing unwanted nodes from a dataflow involves clicking on the little delete icon next to each node you want to remove.  Tedius!

### Check out the [dataflow extractor](https://andrewmiller.online/dataflow-extractor/extractor.html)

### View source code on [Github](https://github.com/AndrewMillerOnline/dataflow-extractor)