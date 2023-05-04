---
title: "Treatise on snapshotting data with CRM Analytics"
tags: [CRM Analytics, Recipes, Business Intelligence]
categories: [CRM Analytics / Tableau CRM / Einstein Analytics, Dataflows & Recipes]
description: Some thoughts on best practices around snapshotting your CRM Analytics data
---

## Introduction

There was a situation where I work recently where two of our snapshot recipes kind of imploded: their runtimes had increased by more than 40% over the course of a month, adding millions of rows and ultimately causing significant delays for the other recipes that were scheduled.  After making some fixes to those recipes, I began thinking about some best practices and considerations to ensure this sort of situation doesn't reoccur.  That morphed and grew into what you now see before you: a treatise on everything snapshot-related.  I hope it's useful!


## Why snapshot your datasets?

Simply put, the added dimension of time can provide incredible value to your data.  It allows your analysis to go one level deeper.  Where once business users were asking "*What is our NPS?*" or "*How much is in the sales pipeline?*", the addition of time via a snapshot can transform those questions into "*How is our NPS changing over time?*" and "*How is our pipeline this quarter compared to the same quarter last year?*".  This superpowers your dashboards: rather than being informational and merely reporting on the current status of a given KPI, they are now facilitating analysis and helping to drive decision making.

That's maybe a bit lofty, and perhaps I'm waxing philosphical, but I hope the point has come across.  The TL;DR is that if a certain metric is worth reporting on, it's likely worth tracking that metric over time too.

## Snapshot recipe vs Watchlist

Before I dive into creating a snapshot recipe, it's worth mentioning that CRM Analytics has a built-in feature that already allows users to track metrics over time: [the Watchlist](https://help.salesforce.com/s/articleView?id=sf.bi_home_watchlist.htm&type=5).  For any number widget you place on a dashboard, users can add that number to their watchlist and CRM Analytics will automatically take a snapshot each day, allowing users to see the trend over time.

There are a few reasons why you might choose to use the Watchlist versus creating a snapshot recipe:
- Only a small subset of users need to see this this metric over time
- As a stopgap if there aren't any resources available to create a recipe

And some reasons why you might choose to snapshot that data instead:
- Provides a standardized source of truth available to all users
- The watchlist exists in its own UI, and can't be consumed in other dashboards or lenses
- Provides more granular control of snapshot time
- Better support if you need to alter the snapshot logic, source data, etc.
- Doesn't rely on a somewhat hidden, power-user feature
- Watchlist is limited to 20 tracked metrics per user

While I think the Watchlist is great and has some good use cases, in general I'd lean towards snapshotting your data in a recipe instead.

## Considerations when snapshotting

Snapshot datasets by their very definition will grow over time as the recipe continually adds more and more rows onto the dataset.  If you're not careful, this can lead to snapshot recipes that eventually explode in runtime, bringing your entire CRMA data infrastructure to a grinding halt.  However, there are four key areas to consider when designign your snapshot recipe that will prevent this explosion: the grain of your snapshot data, the lookback period, your data retention period, and when the snapshot is scheduled to run.

### Grain

The first and most important thing you should consider is the grain of your data, as this can make orders of magnitude worth of difference in how quickly your dataset grows.

Let's use opportunities as an example.  Does your snapshot dataset need to contain every single opportunity?  If you want to see how individual opportunities changes over time, it very well might!  However, if the end goal of your analysis is to see how the opportunity pipeline for an individual sales rep changes over time, then you can probably aggregate your snapshot at the rep level instead.  Continuing this example, if your org has 1,000 reps with an average of 30 opps each, an opportunity-level grain will result in snapshot dataset growth of 30,000 rows per day whereas a rep-level grain would result in only 1,000 rows per day.  This makes a **huge** impact over time.

### Lookback Period

It's also very important to consider the lookback period, or how much data you want to capture in each snapshot.  Do you need to capture the entire opportunity table, or are you really only concerned about current quarter opportunities?  The more you can limit this lookback period, the faster and more reliable your snapshot recipe will be.


### Data Retention Period

The third aspect to consider if your snapshot's data retention period.  In the above paragraph I mentioned limiting how much data you capture in each snapshot.  However, if you never remove any data from your final output the dataset will grow endlessly.  By adding a data retention filter, as I'll show below, your recipe will instead remove data from the output that's no longer relevant.

### Scheduling

Finally, I'd consider when you actually schedule your recipe to run: both in terms of cadence and time of day.  Do you need to snapshot daily, or would a weekly snapshot suffice?  The less frequent your snapshot cadence, the more resistant to implosion your recipe will be.

When it comes to the time of day at which you schedule your recipe to run, my default recommendation would generally always be to snapshot during off hours.  That way, even if your recipe's runtime is rather long, it's less likely to cause a backlog for other business-critical recipes.

Another benefit to scheduling off peak is that you're more likely to capture the entire day's worth of activity.  For example, if you're snapshotting opportunity data at 2PM but a rep created a new opportunity at 4PM, it won't show in the snapshot until the following day.  This might be misleading to users if they expect a given day's snapshot to contain *all* activities from that day.

## Creating the recipe

### General Structure

![the structure of a snapshot recipe](/assets/snapshot_dataset.png)

Snapshot recipes all have the same structure, pictured above.  Here's how each node works:
- **Ingest data to snapshot**.  This is where you're ingest opportunity, quota, or whatever other object you want to snapshot.
- **Lookback period filter**.  As described above, this is where you'll filter out irrelevant data.  Maybe this filters to opportunities created this quarter or to quotes generated today -- it really depends on the business requirements.
- **Aggregate to a higher grain**.  Your snapshot may or may not have this node.  If you can aggregate, I highly recommend doing so.  If your requirements don't allow it, simply delete this node from the recipe.
- **Transform**.  This is the final step before we append the current snapshot data to the existing dataset.  In this step you'll want to generate a "Snapshot Date" variable to represent when the snapshot was taken.  You can also do any other variable cleanup or transformation work needed.
- **Ingest snapshot dataset**.  This node reads in the same dataset that you'll be outputting at the end of the recipe.  By reading in the old dataset then appending a new snapshot on top of the old results, we'll continue to grow out the snapshot over time.
- **Data retention filter**.  As discussed above, at a certain point your old snapshot data is no longer necessary.  Here you can filter on the Snapshot Date column created in the Transform step above, removing snapshot data that's older than your retention window.
- **Append new snapshot to existing dataset**.  Stack the new and old data together to grow your dataset.
- **Output snapshot dataset**.  Output the data for consumption.

### Constructing the recipe

You may have noticed something interesting about the recipe structure above: we're reading in the snapshot dataset, appending new data to it, then _overwriting_ the snapshot dataset.  That means when you go to create this recipe for the first time, that snapshot dataset doesn't yet exist.  As such, you have to create this recipe in two steps.

First you'll create a recipe that includes the nodes from the uppermost "branch" of the screenshotted recipe.  Specifically, you'll ingest your data, filter it, aggregate itm transform it, and then output it.  That creates your snapshot dataset for the first time.

Next you'll add the bottom nodes as well as the append.  Specifically, you'll edit the recipe to bring in the existing snapshot dataset, add the data retention filter, then add the append node.

That's it!

## Conclusion

Hopefully this has made it clear why you might choose to snapshot your CRM Analytics data, and the best way to approach doing so.  If you have any questions, let me know!