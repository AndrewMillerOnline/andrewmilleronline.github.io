---
title: Making an Automated Google Big Query Data Quality Monitor with Slack Alerts
tags: [CRM Analytics, Business Intelligence]
categories: [CRM Analytics, Business Intelligence, Google Cloud Platform]
description: A guide on how to build a data quality regression testing suite for Google BigQuery, complete with Slack alerts
---

## The Premise

Business Intelligence is all about using data to help drive decision making.  Inherent in that definition is the assumption that the data is correct.  In the real world, however, data can occasionally be wrong.  Maybe your ETL script has a small bug.  Maybe an upstream data source messed up.  Maybe a DAG didn't finish fully.

There are tons of possible causes, all with the same end result: your data is incomplete and/or incorrect.  In my mind, the worst possible outcome in this situation is that you unknowingly surface that incorrect data to decision makers.  This might incorrectly influence the decisions and frankly defeat the whole purpose of the business intelligence organization.

So if your data is wrong, you need to know.  And you need to know immediately so that you can take corrective action.  There are a lot of different approaches to doing this, and I'm going to break down how I accomplished it for my team.

## Solution Overview

Our team uses BigQuery to stage our data before ingesting it into CRM Analytics for end user consumption.  Right now we do all our orchestration through cron scheduling of our Python ETL scripts, but we have plans to move to AirFlow for orchestration in the near future.  Building out a cloud-native approach for the data quality testing suite that works with both our current and future orchestration was my top priority.

In a nutshell, the solution works like this:
1. Tests are defined as BigQuery scheduled queries, with one scheduled query per table.  Multiple tests are defined in each script, and a test failure will raise a SQL ERROR()
2. The scheduled query is either scheduled to run at certain times of the day or can be invoked on demand via Python
3. Scheduled query errors are published to a Pub/Sub topic
4. A cloud function subscribes to the Pub/Sub topic, so when it sees an error it then uses an Incoming Webhook to send a message to Slack
5. A Slack app receives the Incoming Webhook and sends the alert to your team's Slack channel, alerting you in real-time to the data quality error.

Tada!

![Screenshot of Slack alert](/assets/slack-data-quality-bot.png)

## Why I like this approach

There are a few reasons why I like this approach that I think are worth sharing.

1. Can be implemented entirely within the Free Tier of Google Cloud Platform, so there's no additional cost.
2. Because it lives entirely within GCP there's no additional software or platform required.
3. The regression tests are simple and easy-to-understand SQL, so anyone on your team can add to the test suite.

## Setup Guide

One of the beauties of this solution is that it's relatively easy to set up: from start to finish, you can have your first test suite running in an hour or two.  (Of course, if you're working at a larger enterprise like mine, dealing with the Slack and GCP IAM permissions for some of these steps will add _days_ to the process...)

### 1. Create Pub/Sub Topic

The first step is to create your pub/sub topic.  Give it a relevant topic id such as "data-alerts".

The "Add a default subscription" checkbox is optional.  You don't need a default subscription for this process - the cloud function will create its own subscription - but it can be nice to have if you want to manually pull your failure event messages for troubleshooting purposes.

Leave the schema and message retention checkboxes unchecked.  Checking the schema option will likely cause this process to fail, and there's no need to retain the pub/sub messages because the failure events are already stored in StackDriver/Logs Explorer.

![Pub/Sub topic creation](/assets/pub-sub-topic-creation.png)

### 2. Create Destination Table

Now switch over to BigQuery. We need to create a new table that will store the results of successful test runs.  Our scheduled query will use SELECT statements, and we have to specify a destination table to store the results of those SELECT statements or we'll get an error.  The upside to this is that this table can serve as a log of all your successful test runs, which is cool.

Give the table a relevant name such as `data_quality_test_results`, and create two columns:

> `test_info (STRING)`

> `etl_timestamp (TIMESTAMP)`

![Destination table creation](/assets/test-table-creation.png)

### 3. Create Scheduled Query

Now that we have a Pub/Sub topic to receive test failures and a destination table to store successful test runs, we're ready to create our first test suite.

Within BigQuery, open a new editor window and construct a test similar to the following.

```sql
  -- This query will run a series of tests to ensure data quality within the new_table table
  -- Each query needs a test_info string formatted as "<TEST_NAME>::<TABLE_NAME>::"
  -- Ensure table isn't empty
SELECT
IF
  (row_count = 0,
    ERROR(test_info),
    test_info) AS test_info,
  CURRENT_TIMESTAMP() AS etl_timestamp
FROM (
  SELECT
    COUNT(*) AS row_count,
    "Table is Empty::new_table::" AS test_info
  FROM
    `my-project-name.test.new_table` )

UNION ALL

  -- Ensure that first name and last name are either both provided or both null
SELECT
IF
  (row_count > 0,
    ERROR(test_info),
    test_info) AS test_info,
  CURRENT_TIMESTAMP() AS etl_timestamp
FROM (
  SELECT
    COUNT(*) AS row_count,
    "Incomplete name fields::new_table::" AS test_info
  FROM
    `my-project-name.test.new_table`
  WHERE
    (NOT(first_name IS NULL)
      AND last_name IS NULL)
    OR (first_name IS NULL
      AND NOT(last_name IS NULL)) )
```

Now obviously this is a simple example, where I perform two tests:

1. A check to ensure that the table has a row count greater than 0.
2. A check to ensure that all the records in this table either have no first_name/last_name or have both a first_name & last_name.

You'll want to tweak the tests to suit your needs.  The possibilities are endless, but you might:
- Check to ensure profit > 0 each day
- Check to ensure that join keys such as employee_id are coming through in the proper format (int versus float, etc)
- Check to ensure the product_name field doesn't contain erroneous or unexpected values.
- Etc.

The most important thing to remember is that the ERROR raised by each test contains a string with the test name and table name, each followed by two colons.  We'll need that string in the cloud function, so you can't omit it.

### Scheduling the query

Now that the query has been created, you need to schedule it.  Press the Schedule button at the top of the editor window and create a new scheduled query.

First, give your query a name such as "Data Quality Test: table_name".  Following a set nomenclature like this will help you easily find the test suite you need if you have a lot of scheduled queries.

Next, choose how often to run the test.  If your tables are refreshed every day at the same time, you can easily schedule the query to run a minute or two later.

If your tables aren't refreshed on a predictable cadence, you can also choose the "On-demand" option from the Repeats dropdown.  At that point, you can either use the web ui or the [Data Transfer API](https://cloud.google.com/bigquery-transfer/docs/reference/datatransfer/rest/v1/projects.locations.transferConfigs/startManualRuns) to initiate your tests when needed.

Under the Destination for query results section, select the destination table you created in Step 2 above.  Make sure the "Append to table" radio button is selected so that the table continues to aggregate all successful runs.

Finally, under the notification options section, enter the pub/sub topic you created in Step 1 above.

### 3. Create Slack App

It's time to create our Slack app.

Go to the [Slack Apps page](https://api.slack.com/apps) and click the Create New App button.  Choose the 'From scratch' option, then give your app a killer name and choose the Workspace where you want this app to live.

Once the app is created, go to the Incoming Webhooks page and activate the feature.  Scroll to the bottom of the page and click the "Add New Webhook to Workspace" button.  Now select which channel your App should post alerts to and click 'Allow'.

You'll be returned to the Incoming Webhooks page where you can now see a URL that starts with `https://hooks.slack.com`.  You'll need this URL for the next step.

That's it!  Before moving on to the next step, you might want to give your app a sweet icon and meaningful description under the Basic Information section.

### 4. Create Cloud Function

We're almost done!  We now have automated data quality tests running, which publish to a pub/sub topic when bad data is found.  We also have a Slack app that will alert us to those test failures.  We just need a relay to tie them together.

Head back to your Google Cloud console and open up Cloud Functions.  Create a new function.

Set environment to 1st gen and give your function a name, such as "data-quality-slack-notification".

Now under the Trigger section, select "Cloud Pub/Sub" as the trigger type and select the pub/sub topic you created earlier.  Hit Save, then Next.

Cloud Functions support a number of runtimes, but I made this in Node.js so you'll want to select Node.js 16 as the runtime.

Enter the following code into the package.json file:

```json
{
  "name": "data-quality-slack-notification",
  "version": "0.0.1",
  "dependencies": {
    "@google-cloud/pubsub": "^0.18.0",
    "@slack/webhook": "^6.1.0"
  }
}
```

Now switch back to the index.js file, edit the entry point to "subscribe" and enter the following code:

```javascript
const {IncomingWebhook} = require('@slack/webhook');
const SLACK_WEBHOOK_URL = "<YOUR WEBHOOK URL FROM STEP3>";
const webhook = new IncomingWebhook(SLACK_WEBHOOK_URL);

// subscribe is the main function called by Cloud Functions
module.exports.subscribe = (event) => {
  const eventData = getMessageData(event.data);

  //Only send Slack messages on failures
  if(eventData.errorStatus){
	console.log(eventData.errorStatus.message);

  	// Send message to Slack.
  	const message = createSlackMessage(eventData);
  
  	webhook.send(message);
  }
};

// Transform pubsub event message to a string
const getMessageData = (data) => {
  return JSON.parse(Buffer.from(data, 'base64').toString());
}

// Create the Slack message
const createSlackMessage = (eventData) => {
  let messageParams = eventData.errorStatus.message.split('::');

  let message = {
	"blocks": [
		{
			"type": "header",
			"text": {
				"type": "plain_text",
				"text": ":warning: Data Quality Test Failure",
				"emoji": true
			}
		},
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": `The test _${messageParams[0]}_ has failed on table _${messageParams[1]}_. <https://console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1s<YOUR PROJECT ID>!2s<DATASET NAME>!3s${messageParams[1]}&project=<YOUR PROJECT ID>|Open table in GBQ>`
			}
		}
	]
};

  return message;
}
```

There's two things you'll need to edit in the code:
1. First, put your Slack webhook URL into line 2.
2. Then, update line 43 with your GBQ project id and dataset.  That section inserts a link into the Slack message to easily open the offending table in the GBQ web UI, so it needs to know which project and dataset your table is in.

That's it!  Go ahead and deploy the code then give it a few minutes for the function to build.

### Let's test it out

Now that everything is configured, let's run your first test suite.  Go back into the scheduled queries interface and open your scheduled query.  Click the "RUN TRANSFER NOW" link at the top of the page and initiate a one time transfer.

If everything is good with your data, then absolutely nothing will happen!  We don't want to make too much extra noise, so the cloud function will simply exit if there are no errors.

However, if there are any data errors, you should see a new Slack alert pop up about a minute after initiating the transfer run (it takes about a minute in my experience for the transfer runs to complete).

![Our bot works](/assets/dq-bot-working.png)