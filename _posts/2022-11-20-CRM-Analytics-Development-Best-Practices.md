---
title: "Tableau CRM Development Best Practices"
tags: [CRM Analytics, Tableau CRM, Einstein Analytics, SAQL, Development]
categories: [CRM Analytics / Tableau CRM / Einstein Analytics, SAQL]
description: Applying lessons and best practices from software development to Tableau CRM
---

Coming from a software development background, when I moved into the world of CRM Analytics I immediately noticed that certain best practices from software development were just as applicable when building dashboards.  These aren't hard and fast rules, but following these recommendations will enable your team to work more cohesively, with fewer bugs, and less technical debt.

## 1.  Clicks over code - don't use SAQL unless you have no other choice

Ok, I'll admit that this first one isn't really a software development thing, and more of a best practice for Salesforce at-large.  However, I agree with this recommendation so much that I wanted to include it anyway.

There are a few reasons why you should avoid using SAQL when building your visualizations:
-  The drag and drop query builder interface is much easier for people to learn than SAQL syntax.  This is true for your end users, who might want to customize their visualizations, as well as for newer CRMA developers.  Keeping things simple will make everything more approachable.
-  Building on that, if you use SAQL to generate a table, users will no longer be able to click a column header to sort.  This creates extra work for you, as you'll have to create custom bindings to enable table sorting.
- Moreover, if you use SAQL users can no longer easily Explore the visualization and modify the filters.  In our org, many users want to create a custom lens showing data for a specific sales team or group of sales teams.  But if the query has SAQL, that options becomes more difficult.

Ultimately, the use of SAQL makes self-serve analytics more difficult.  We don't want sales people wasting time learning SAQL syntax.  Rather, we want them to be able to quickly use a visualization to identify potential profit, and then act upon those insights.

## 2.  Use Version History descriptions for all dataflow, recipe, and dashboard changes

Across the software industry, most teams use git for version control.  Git has a lot of benefits: it allows you to easily separate code changes and revert back to a specific change, it allows you to see who changed what code and when, it has branches that allow multiple simultaneous workstreams to take place, and many more.

CRMA's native version history isn't nearly as robust as git.  It doesn't allow for different branches, it doesn't provide merge conflict resolution, and you can't revert the changes of a specific commit if it's not the most recent (in other words, you can revert back to the version published at a specific point in time, but you can't undo a save from within the stack of changes).

That said, every single CRMA developer should be using this feature!

Version History lets you see who changed the dashboard, recipe, or dataflow most recently.  Whenever you make a change, you should also provide a description in the Version History describing the change.  CRMA doesn't have a built-in diff function - although you can download the two JSON files and manually diff them - so this description is huge in helping your teammates understand what you changed and why.

You can also revert back to a specific state using Version History, which can be helpful to quickly undo one or more saved versions should you find that they contained bugs or breaking changes, for example.

The most powerful aspect of Version History only works with dashboards, and it lets you distinguish one version of the dashboard as "live" to your end users, while maintaining a separate stream of versions in "draft" mode visible only to selected dashboard publishers.  This is super powerful if you're making small changes to a dashboard but don't want to introduce those changes to users yet.  By adding testers to the publishers list, they can do QA and UAT against the draft version before ultimately setting that version live to the broader audience.

## 3.  When you have to use SAQL, write code comments

As much as I want to always adhere to point number one above, it's not always possible.  Sometimes there is no choice but to write SAQL in order to achieve the intended visualization.  When you reach that point, it's important to shift to the mindset of someone who is writing code (because you are!).  That means commenting your code.

Code comments are meant to document the *why* rather than the *how* behind a piece of code.  You should assume that the next person reading this code has a better understanding of CRMA and SAQL than you, but a worse understanding of the business rules.  The reason for this is simple: while you might have a solid understanding of the data structure or business rule nuances behind the item you're working on right now, if you come back to it a year down the road chances are you won't remember all of the intricacies and nuances.  Moreover, if a new team member has to work on that code at some point, they likely won't have any of that historical context.  So do your future self and future teammate both a favor and comment your code.

## 4.  Use logical, explanatory names

I feel like this item should be self-explanatory, but the truth is that it's not.  I don't know how many times now I've seen queries in the dashboard editor with names like *lens_1* or *lens_1_1*.  It's lazy and unhelpful - how is anyone supposed to know what lens_1 does?  This is the sort of practice that leads to bugs and technical debt.  Being proactive in your variable naming can save you a lot of time down the road.

Here are the three areas where you can and should use logical, explanatory names: your query names, SAQL variable names, and node names in dataflows and recipes.

### Query Names
Rather than letting the query edit default the name to *lens_1*, try to establish a naming convention.  I like to prepend each query with the intended output.  For example, tbl_XXX for tables, kpi_XXX for numbers, and chart_XXX for vizzes.  The XXX portion gives more context on what's being presented.  For example, kpi_win_rate and tbl_win_rate_hist would be used to show a large win rate KPI and a table of historical win rates, respectively.

> When it comes to naming conventions, much is based on personal preference.  You don't necessarily need to prepend with tbl, kpi, and chart.  Those are just what I like.  Nor do you need to use snake case; again, that's again just my personal preference (snake case, in case you're not familiar with the term, is when you use underscores to separate word.  ie: tbl_win_rate is snake case, whereas tblWinRate is camel case).  The key is that you choose a convention, *any* convention, and **stick. with. it.**  This reduces confusion and increases synergy across your development team.
{: .prompt-info }

### SAQL Variable Names
In CRM Analytics' SAQL editor, the default query variable is q.  as in:
```
q = load "dataset";
q = foreach q generate 'something';
```
{: file='SAQL'}

q is a pretty random and arbitrary choice.  I can only assume the q is short for query.  But it doesn't tell you a single thing about what the query is or what it does.  As you write more and more complex SAQL, chances are you'll start to load and cogroup multiple datasets.  When you get to that point, how do you really tell them apart?  With logical names, of course!

Here are two small snippets of SAQL.  Which is easier to understand?

Number One:
```sql
q = load "dataset";
q = filter q by 'isWon' == "true";
q1 = filter q by 'isClosed' == "true";

q = group q by all;
q = foreach q generate "0" as 'index', count() as 'count';

q1 = group q1 by all;
q1 = foreach q1 generate "0" as 'index', count() as 'count';

q = cogroup q by 'index' full, q1 by 'index';
q = foreach q generate first(q.'count')/first(q1.'count') as 'win rate';
```
{: file='SAQL'}

Number Two:
```sql
q = load "dataset";
q_won = filter q by 'isWon' == "true";
q_all = filter q by 'isClosed' == "true";

q_won = group q_won by all;
q_won = foreach q_won generate "0" as 'index', count() as 'count';

q_all = group q_all by all;
q_all = foreach q_all generate "0" as 'index', count() as 'count';

combined = cogroup q_won by 'index' full, q_all by 'index';

combined = foreach combined generate first(q_won.'count')/first(q_all.'count') as 'win rate';
```
{: file='SAQL'}

Now obviously those are pretty simple examples, but I think the latter is easier to understand due to variable names that are logical and explanatory.  Again there are arbitrary choices being made here: you don't need to prepend "q_" to these, nor do you need to differentiate with the "combined" name.  Again, the most important thing is choosing a convention and sticking with it across your team.

### Dataflow/Recipe Node Names

Dataflows and recipes can often get very long and very messy.  Using logic node names reduces confusion and speeds up work during future iterations.  This is also another instance where you can apply standardized naming conventions; for example, naming all your recipe join nodes something like `[Inner/Left/Lookup/etc] Join <source 1> to <source 2>`.

## Summary

To wrap this up, I'll share a small anecdote from one of my coworkers who works as a data engineer supporting our CRMA development team.  He told me that he could quickly tell who on the team wrote a given Python script based on the syntax and variable naming.  He said that all of the data engineers had their own unique development fingerprint that they'd leave behind in their scripts.

Despite my many recommendations towards establishing convention above, I don't necessarily think that's a bad thing.  We should never get so formal and formulaic that we stifle creativity.

However, when it comes to production code that is maintained by a team of developers -- and **especially** if that team ever experiences turnover or growth -- convention can be the rosetta stone that helps us all get up to speed quickly, while reducing misunderstandings, bugs, and technical debt.