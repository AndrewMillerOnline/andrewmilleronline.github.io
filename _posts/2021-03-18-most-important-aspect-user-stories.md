---
title: "The Most Important Aspect of Agile User Stories"
tags: [Development, Agile]
style: fill
color: info
description: Thoughts of writing effective user stories that will expedite your Agile development team
---
# The Most Important Aspect of Agile User Stories

Good communication is essential to any team, and for agile development teams, writing effective user stories is one of the most important means of communication we can engage in.  User stories are simple enough in concept, and boil down to something like this:

> As an X user, I want Y feature, so that I can accomplish Z task.

However, too often I find the third element, the why, is muddled or even excluded altogether.  Let's examine three versions of the same user story to see how big of a difference the why aspect makes.

## Version One: No Why
> As a content manager, I want to add all countries to a record at once.

This story seems straightforward, and is most likely detailed enough that the team's developer can get started right awya on feature development.  However, there's no *why* and therefore a misunderstanding is quite possible.

![Adding a country by typing out the name and using autocomplete](/assets/add-country-single.png)

For example, the screenshot above provides one possible interpretation of this story.  The user can type in country names, they get some nice time-saving autocomplete, and can then save multiple countries with one click of the save button.  Story done!  Or is it?

## Version Two: The Redundant Why

The first story didn't include a why, so let's add one:

> As a content manager, I want to add all countries to a record at once, so that I can associate multiple countries to a record

This story still doesn't get us where we need to be, because the goal is fairly redundant; in essence, it saying "I want to add countries to a record so that I can add countries to a record."  It misses the fundamental essence underlying the customer's need.

## Version Three: Including the Goal
> As a content manager, I want to add all countries to a record at once, so that I can quickly create records with 180-190 countries.

The key difference, the why, helps to explain how the user will actually be utilizing this new functionality.  Not only will they be associating multiple countries to a record, but they'll be associating hundreds of countries to each record.

Now we know why they truly want this feature: it's not just about the basic functional requirement of associating multiple countries to a record, or even doing it all in one submission or database transcation.  Instead, our user will be adding so many countries to each record that being able to add them all in one button click compared to hundreds represents a **significant** time savings.

![Adding countries via multiselect checkboxes with a select all](/assets/add-country-multiple.png)

As a result of this added clarify, we now arrive at a solution that meets the customer's need.