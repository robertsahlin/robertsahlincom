---
title: "BigQuery data error reporting"
date: 2021-02-06T13:50:33+01:00
draft: true
tags: ["BigQuery", "Validation", "Observability"]
---

This is part 2 in a series of data observability in GCP using only GCP native services (i.e. no third party tools or libraries). You find the first part [here](/validate-and-monitor-your-bigquery-data/).

If you followed part 1 you now have:

- scheduled data quality checks as SQL queries using BigQuery scheduled queries
- email alerts when mentioned queries report data quality errors
- logging of data quality errors
- a basic user defined data quality error metric
- a basic dashboard you can use to set alert thresholds in cloud monitoring

That's awesome. But now you want to take action on the errors in a structured way and even if e-mail is great it isn't the best place to get a sense of how frequent the error is and its status (raised, acknowledeged, resolved, etc.), especially if you are a data engineering team working on fixing alerts. So how can you accomplish that on GCP?

We will take advantage of the [cloud monitoring](https://console.cloud.google.com/monitoring) features and level up our data quality monitoring.  

Let's pretend we have a use case that is a data export from Google Analytics 360 to BigQuery. We want a query that checks that the data has been exported to BigQuery as expected and alert us if not.

First, we will add extra labels to our bigquery_validation_metric to enable better filtering in cloud monitoring. Go back to the user defined metrics in [legacy log viewer](https://console.cloud.google.com/logs/metrics) and click on "edit" of the bigquery_valiation_metric you created in the previous post. Add a regular expression to your "error_message" label to extract the actual message that will be used to create different alert policies for different data validation queries.

```
validation error: (.*)
```

Then add a label "job_name" of type String and field name:

```
protoPayload.metadata.jobChange.job.jobName
```

Do the same with a "query" label defined by field name:
```
protoPayload.metadata.jobChange.job.jobConfig.queryConfig.query
```

Update your metric and go to cloud monitoring. In order to get [alerts](https://console.cloud.google.com/monitoring/alerting) when BigQuery data quality errors are logged you need to [create a policy](https://console.cloud.google.com/monitoring/alerting/policies/create). Here's the [official guide/documentation](https://cloud.google.com/monitoring/alerts/using-alerting-ui#create-policy) but I will give some extra guidance for this particular use case.

1. Add condition
One limitation using cloud monitoring for data quality monitoring is that the alert period only support up to 24 hours. This has an implication if you can't close an alert within 24 hours since even without action it will self heal and the alert will be classified as recovered. Hence, you will have to run the check at least daily and write your SQL query such that it checks data quality not only for the last day but also a period before that (how long period depends on how long you want an alert on data quality issue that isn't fixed), so an error that happens day X should also trigger an alert when running the data validation query at day X+5 if the data quality issue isn't repaired.

So, name your condition and set target metric:

```
Resource: BigQuery Project
Metric: logging.user.data-quality-error
Filter: GA 360 export failed
Group by: error_message, job_name, query
Aggregator: sum
Period: 5 minutes
Trigger: any timeseries violates, condition is above, threshold 0, for most recent
```


2. Who should be notified
Cloud monitoring offers a wide range of notification options. We'll use email in this case.

3. What are the steps to fix the issue
Here you can describe what steps should be taken to address the data quality issue. It could be a backfill, or deleting duplicate rows, or something else.

# An alert fires, now what?
To test your set up, let's manually trigger an error by running the query below in BigQuery:

```
SELECT  ERROR("validation error: GA360 export failed")
```

When an alert fires, you will get a notification in the notification channels you've specified. You will even get links to the incident and the logs.

If you view the incident you will see some background containing both the job name and the query that cause the alert. Now you can acknowledge the issue by clicking the menu in the top, that will timestamp when the issue was opened. While fixing it you can add annotations to the issue to inform your team members that you are working on it and what steps you take to repair it. When you have addressed the issue it is time to close it, you do it in top menu the same way as you acknowledged it. That will timestamp the issue when it was closed and you get a value on how long it was open, i.e. duration.

a

