---
title: "Treatise on snapshotting data with CRM Analytics"
tags: [CRM Analytics, Recipes, Business Intelligence]
categories: [CRM Analytics / Tableau CRM / Einstein Analytics, Dataflows & Recipes]
description: Some thoughts on best practices around snapshotting your CRM Analytics data
---

## Why snapshot your datasets?

## Snapshot dataset vs Watchlist

## Considerations when snapshotting

Snapshot datasets by their very definition will grow over time as the recipe continually adds more and more rows onto the dataset.  If you're not careful, this can lead to snapshot recipes that eventually explode in runtime, bringing your entire CRMA data infrastructure to a grinding halt.  However, there are three key areas to consider when designign your snapshot recipe that will prevent this explosion: the grain of your snapshot data, when the snapshot is scheduled to run, and your data retention period.

### Grain


### Scheduling


### Data Retention


## Creating the recipe

