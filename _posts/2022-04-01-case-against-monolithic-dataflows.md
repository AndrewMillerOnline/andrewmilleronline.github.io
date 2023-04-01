---
title: "The Case Against Monolithic Dataflows"
tags: [Tableau CRM, Einstein Analytics, Dataflow]
categories: [CRM Analytics / Tableau CRM / Einstein Analytics, Dataflows & Recipes]
description: Some thoughts on best practices around architecting dataflows and recipes
---

## Background
My project at work this quarter has been tackling various aspects of technical debt in our TCRM environment.  Under that banner, one specific initiative that I spearheaded was to break down our main dataflow into smaller pieces.  There are many compelling reasons why I wanted to make that change, and I believe that those reasons may also translate to your use case, as well.

## What's a monolith?

Monolith isn't a term often used in the Tableau CRM realm, but it's quite common in software engineering at large.  In fact, [Professor Wikipedia](https://en.wikipedia.org/wiki/Monolithic_application) has the following definition:
> In software engineering, a monolithic application describes a software application that is designed without modularity.  Modularity is desirable, in general, as it supports reuse of parts of the application logic and also facilitates maintenance by allowing repair or replacement of parts of the application without requiring wholesale replacement.

It's with that definition in mind that I define the following characteristics of a monolithic dataflow:
- It takes 90+ minutes to run
- It registers a majority of your datasets
- An error near the end of the dataflow would leave your users with unacceptably stale data

If you have any dataflows that meet one or more of the criteria above, they're probably monoliths.  But...so what?  Why does that matter?

> This post applies to recipes too, but for simplicity's sake I'll just use the term dataflow throughout.
{: .prompt-info }

## The dangers of a monolith

### Danger #1 - They slow down development of new features

Let's say you want to add some additional functionality to an existing dashboard, which requires computing a new field in the dataflow.  If that dataflow takes 90+ minutes to run, you're stuck twiddling your thumbs unable to actually use the new variable for almost 20% of the workday while you wait for your monolithic dataflow to complete.

Smaller, more modular dataflows run faster, which ultimately lends itself to a quicker, more nimble, and more iterative development team.

### Danger #2 - They're harder for large teams to coordinate changes

At Indeed, we support one of the largest Tableau CRM implementations that I'm aware of.  To manage this, we have a team of 6 TCRM developers plus a manager and PM.  This means that at times, we have 8 people modifying various dashboards, lenses, recipes and dataflows.  And while Salesforce released version control for Tableau CRM, it's still a far cry from full version control like you find with Git, Subversion, etc because it has no conflict management.

So what happens when one of my teammates and I both modify a dataflow at the same time?  Whoever saves it last wins.  Our team has implemented some excellent control systems to help us manage who "owns" an asset and has priority for modifying it, but accidents still happen.  With monolithic dataflows, these accidents are more likely to occur because there are fewer dataflows.  By contrast, if you take your single monolithic dataflow and split it into 2, 3, 8, or however many component parts, you're less likely to encounter conflicts because there are more assets.

Obviously, the importance of this factor depends on your team's size.  If you have a small implementation with only 1-2 developers, you can likely communicate easily enough to prevent conflicts.  However, as team size increases -- and especially if you have a geographically distributed team working from various timezones -- the importance of shifting away from a monolithic architecture increases significantly.

### Danger #3 - They lack scheduling nuance, leading to stale data

Salesforce provides some decent flexibility with regards to scheduling dataflows, allowing you to create a schedule that triggers every X minutes, X hours, etc.  But not all data has the same velocity, so when it comes to dataflow scheduling, one size does not fit all.

To illustrate, imagine a Salesforce environment where sales reps are creating and working Opportunity records all day.  You will probably want to sync the Opportunity object multiple times per day, so that reps always see the latest info.  At the same time, say you have a User object that stores records about your sales reps.  At many companies, you probably only onboard once per week, so you really only need to refresh the User object once or twice per week.

In that scenario, you have data from two objects with two very different velocities.  Put together in one dataflow, you'll encounter one of two results:
- If you run the dataflow every 4 hours, you'll be processing the User object too frequently.  Outside of the first run after Monday's onboarding, the dataflow will just process the same records every time.  Inefficient.
- If you run the dataflow once per week, after the User records are updated, your opportunities will be incredibly stale.  Sales reps won't want to use your dashboard because it's not providing real-time insights.

Now obviously that was a somewhat simplified example, but I hope it makes the point.  Different data have different velocities and different importance to the business.  They need to be broken into separate dataflows based on those factors.

### Danger #4 - Failed runs lead to even more unacceptably stale data

Imagine your data sync runs at 8am daily, followed shortly thereafter by your multi-hour monolithic dataflow.  Should that dataflow fail - and I've certainly experienced my share of random transient errors - even if you catch the error quickly and re-run the dataflow successfully, you could now be in a situation where it's 11 o'clock or later and your users are just now getting fresh data for the day.

With smaller dataflows which have schedules aligned to their replicated object's sync schedules (or better yet, are run on demand), you can be assured that your users will get the freshest data available, as soon as it's available.  And when it comes to enabling data-backed business decision making, the importance of fresh data cannot be overstated.

## The solution: smaller, modular dataflows

Now that I've laid out the dangers of a monolith, I hope the solution has become clear.  You should probably break your dataflow into smaller chunks.  How exactly you decide to split your dataflow should be based on a few factors:

- How often the records being processed in the dataflow update
- What time (time of day and day of week) the records being processed update
- Interdependencies, where dataset X might rely on one or more source objects
- How critical a particular dataset is to business decision making
- The size of your team and how often multiple developers need to modify the same dataflow definition
- How close you are to hitting the 60 flow per 24 hour limit