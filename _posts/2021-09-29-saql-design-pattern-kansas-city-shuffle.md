---
title: "SAQL Design Pattern: The Kansas City Shuffle"
tags: [Tableau CRM, Business Intelligence]
style: fill
color: primary
description: 
---

# Background
When building dashboards, I frequently find myself needing to reshape data.  Maybe it's in a long format, but visually would make more sense as a wide table.  Maybe it's the other way around.  In fact, I come across this situation so often that I've devised design patterns around each scenario.  Today I'll talk about how to transform from long to wide, a transformation that I've nicknamed the Kansas City Shuffle.

# Sample Data

Let's say we have a sample dataset called Variables and it contains two columns: Variable and Value.  Here are some sample rows:

| Variable| Value |
|---|---|
| Name | Andrew |
| Age | 34 |
| Gender | Male |
| Location | Illinois |

While that shape can make sense from a data architecture perspective (depending on various factors), it won't really look good in a dashboard.  Instead, we'd probably prefer the data to appear this way:

| Name | Age | Gender | Location |
|---|---|---|---|
| Andrew | 34 | Male | Illinois |

It's an easy fix with the Kansas City Shuffle!

# The Design Pattern

```sql
---First we need to load and filter our dataset to get the initital table above
q = load "Variables";
q = filter q by <<whatever>>;

--- For each row that we want to transform to a column, we'll need a separate variable
q1 = filter q by 'Variable' == "Name";
q1 = foreach q1 generate "0" as 'index', 'Value';

q2 = filter q by 'Variable' == "Age";
q2 = foreach q2 generate "0" as 'index', 'Value';

q3 = filter q by 'Variable' == "Gender";
q3 = foreach q3 generate "0" as 'index', 'Value';

q4 = filter q by 'Variable' == "Location";
q4 = foreach q4 generate "0" as 'index', 'Value';

/* Now we can cogroup our four queries together to get them in one row.
 In theory, you should be able to cogroup all four in one call as such:
   res = cogroup q1 by 'index' left, q2 by 'index', q3 by 'index', q4 by 'index';
 However, while this is syntactically correct, in my experience it doesn't work.
 Instead, we'll need to do 3 cogroups! */

res = cogroup q1 by 'index' left, q2 by 'index';
res = foreach res generate q1.'index' as 'index', first(q1.'Value') as 'Name', first(q2.'Value') as 'Age';

res = cogroup res by 'index' left, q3 by 'index';
res = foreach res generate res.'index' as 'index', first(res.'Name') as 'Name', first(res.'Age') as 'Age', first(q3.'Value') as 'Gender';

res = cogroup res by 'index' left, q4 by 'index';
res = foreach res generate first(res.'Name') as 'Name', first(res.'Age') as 'Age', first(res.'Gender') as 'Gender', first(q4.'Value') as 'Location';

--- That's it!  Our table is now in a wide format.

```

# FAQ

### Why include 'index'?
We need something to serve as a key when cogrouping the queries together.

### Why is the index a zero string "0"?
Believe it or not, if you generate 0 as 'index' and use that to cogroup, SAQL will return an error due to the Numeric type.  So I put it in a string.  The value of zero is arbitrary.

### Why use the first() function?
Cogroup will automatically group based on the key provided, so we need to use aggregate functions.  This can complicate things, especially if our table had included information for more people.  If it has 2 people, we can use first() and last() to distinguish.  If it has 3+, things get even more complicated.

### Isn't there an easier way to do this?
Not that I know of.  If you know another way, I'd love to hear!