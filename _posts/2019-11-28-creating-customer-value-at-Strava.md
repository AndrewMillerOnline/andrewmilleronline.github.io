---
title: "Creating Customer Value at Strava"
tags: [Case Study, Entrepreneurship]
style: fill
color: secondary
description: Some thoughts regarding Strava's way forward.
---

## Background

Strava is a growth stage startup, which means they have one goal: increasing their customer base.  While they have been hugely successful at increasing their user base--to the tune of 1M new users each month--that hasn't been translating into paying Summit customers.  As a long-time and entrepreneurial-minded Strava user (not customer), my take on *why* is simple: they're not creating enough value for the customers.

## How Strava Can Increase Value

Strava has a pretty simple model, which includes a set of free features available to all users and a set of paid features available only to their Summit customers.  Increasing the customer base can therefore take one of two approaches: add more Summit features (increase value) or figure out how to better communicate the existing value.  While simple, it's certainly not easy, and figuring out *which* features to add is where Strava needs strong product management.

## Value Capture and API

One area of improvement is in value capture, specifically surrounding Strava's API.  Now, APIs are great.  They help to increase the ecosystem around a product, and can even become a full-fledged revenue producing product in and of themselves.  Moreover, I believe that Strava's API represents a strategically significant product moving forward, especially considering the recent trends in the wearable fitness industry (case in point: [Google acquiring FitBit](https://www.cnbc.com/2019/11/01/google-to-acquire-fitbit-valuing-the-smartwatch-maker-at-about-2point1-billion.html)).

That said, Strava could be utilizing its API to identify and vet potential product enhancements.  Just take a look at [StravaBestEfforts.com](http://stravabestefforts.com), a site that lets users track their Best Efforts over time.</p>

![](/assets/strava-best-efforts.png)

Best Efforts are purely a creation of Strava: they are a programmatic calculation of how quickly you've run a specific distance, based on your GPS data.  Strava simply displays your best Best Effort for each distance on your profile, and shows you a little medallion on an activity where you've clocked a new top three best effort.  But what about this simple user story?  "As an athlete, I want to track the improvement in my Best Efforts over time."  Strava doesn't have anything resembling this functionality, which is no doubt how StravaBestEfforts came into being.

So what's the big deal with this?  Simply put, this is value that Strava isn't capturing from its user data.  All that StravaBestEfforts does is retrieve user data from Strava's API and then sort that data into a table.  It doesn't even have graphs, it's just an ugly table!  (There are some graphs on the Stats page, but they're pretty bad.)  The big thing here is that it also has a paid account, for which it charges $2.99/month.  Think about that.  It's providing a very simple data visualization tool (plus a few additional features with Premium, such as weather tracking) for 60% of Strava's Summit 3-pack price.

While I don't know how many users are paying for this functionality, this is still an incredibly strong demand signal.  Even if only a handful of users pay for it, those are users that had to 1) actively search for, seek out, or somehow find this site;  2) provide permission to link their Strava account;  3) shell out their Paypal or credit card information.  The conversion rate would skyrocket if all of those steps were conveniently rolled into a Summit pack, let alone if the data were presented in a more attractive fashion.

## Generalizing API value capture

While my conclusion is that Strava needs to implement this best effort tracking functionality, the truth is that's not based on empirical evidence.  I'm not a product analyst at Strava, and I don't have access to their data.  I also don't have insight into the number of paying customers at Strava Best Efforts.  So how can we make this decision more empirical, and also generalize it to extend to products other than Strava?

1.  Implement tracking analytics on your API if you don't already have them.
2.  Analyze those metrics for a given period of time, sorting by your top API clients, and then go find, use, and understand their product.  Figure out what they're doing with your API data to determine if they're *adding value* to your product/ecosystem or if they're *siphoning value*.  I would argue that StravaBestEfforts is siphoning value that Strava should be capturing itself.
3.  Once you've identified a value siphoning API client, analyze the API calls to determine the number of users, and try to determine how many of them are paying.  These figures should provide a decent basis for determining the potential upside for implementing the feature in your own product.  In the case of StravaBestEfforts, free users are limited to 180 days of Best Effort comparison, so it might be possible to differentiate between free and paying users by the activity date ranges included in the API calls.

## Conclusion

This is getting long, so I'll wrap up for now.  The bottom line is that Strava needs to increase value for its customers, and the most surefire way to do that is by adding more paid features.  A smart product leader is one who uses data to drive their decisions, and in this case Strava could be utilizing their API data to vet demand for new features to ensure ROI for their development efforts.