---
name: Viromob
tools: [Node.js, Angular, Ionic, Firebase]
image: \assets\viromob-logo.png
description: Mobile-first giveaway app
order: 7
---

## Overview

Viromob is a web-based business that I worked on between 2019 and 2020 as a solo founder.  Viromob is intended to be a tool for small business owners to promote their business through social-media giveaways.  With Viromob, entrepreneurs can easily host a product giveaway where customers earn entries through social actions, such as following the brand on Instagram.  Additionally, Viromob has an accompanying platform where users can find new giveaways to enter, see their current giveaways, and more.

## The Tech Stack

![Viromob Architecture](/assets/viromob-architecture.png)

Viromob contains two applications, the Giveaway Entry app and the Sponsor Dashboard app, as well as a Marketing Site.

**Giveaway Entry App:**
* [https://app.viromob.com](https://app.viromob.com)
* Progressive Web App
* Coded with Angular 9 and Ionic Framework
* Mobile-first design

**Sponsor Dashboard:**
* Angular 9 website

**Marketing Site:**
* [https://www.viromob.com](https://www.viromob.com)
* Static landing page built with Webflow
* Content marketing blog powered by Webflow CMS
* Integrates with Zapier and MailChimp

## The Backend

One of the most interesting parts of programming Viromob was working with Google Cloud Firestore and Cloud Functions to create a very powerful headless backend.  There are some semi-complex business rules related to a giveaway ending and the winner being drawn, notified, and confirmed that are enforced through Cloud Functions: a blog post detailing this is upcoming.  This also tied in nicely with beautiful transactional emails sent through MailJet.