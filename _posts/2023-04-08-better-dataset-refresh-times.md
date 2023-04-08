---
title: Better Dataflow Refresh Times
tags: [CRM Analytics, LWC]
categories: [CRM Analytics, LWC]
description: CRM Analytics LWC that adds clarity to your dashboards
---

## The Problem

Companies use analytics because data has the power to improve decision making: both in terms of quality and speed.  But to get to that glorious end state, users must be able to understand and trust your data.  A large factor of that is understanding the data freshness.

>Is this aggregated sum of opportunity amount going to include opportunities created 3 minutes ago?  I thought the data was live.  Oh wait, it's actually 24 hours old??  That's useless!

You don't want those sorts of thoughts going through your users' heads!

CRM Analytics dashboards have a great built-in feature to help solve this problem.  At the top of each dashboard there's a line that displays the latest refresh time for the data within your dashboard.  But in truth, it's not so great if your dashboard pulls from two or more datasets -- especially if those datasets have different refresh times or cadences.  In those instances, the dashboard's refresh time might actually be confusing your users more than helping them.  

## The Solution

To help make it more clear to your users when their data was updated, I used CRMA's embedded LWC functionality along with the Wave API to create a small component that you can throw into your dashboard.  Give it a dataset ID, and it will display the dataset's last refresh date.  Even better: if you enter the refresh cadence, it will also show the number of hours until the dataset refreshes again.

Some notes:
- If the next refresh is within the next hour, it will show the countdown in minutes
- If the next refresh is running late (dataflows/recipes are backed up, we've all been there), it will omit the time until next refresh
- The dates are timezone-aware based on the user's browser

## Preview

![dataset refresh lwc demo](/assets/dataset_refresh_demo.png)

In the sample dashboard above, I recreated a very typical usecase: a dashboard with two data tables powered by datasets created from different dataflows, and hence different Last Refreshed dates.

The widget directly underneath each table is our LWC!

Note how the right hand widget is also configured to display a countdown until the next refresh.  This option is perfect for regularly scheduled recipes/dataflows that you know will run every X hours.

The LWC includes a few basic configuration options required to make it work: first, you must provide a dataset using either the ID or the API name.  You must also provide a user-friendly label for that dataset such as "Opportunity" or "Finance Revenue".

Then there are some optional configs, such as: the refresh cadence if you want to show a countdown until the next refresh, and font size/color options that allow you to customize the display of the LWC to match your overall dashboard theme.

![lwc-config-options](/assets/dataset_refresh_config.png)

## Source Code

[Full source available on GitHub here](https://github.com/AndrewMillerOnline/crm-analytics-lwc-playground/tree/main/force-app/main/default/lwc/datasetRefreshDisplay).