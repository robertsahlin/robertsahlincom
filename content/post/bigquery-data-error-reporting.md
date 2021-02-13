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

First, we will add extra labels to our bigquery_validation_metric to enable better filtering in cloud monitoring. Go back to the user defined metrics in [legacy log viewer](https://console.cloud.google.com/logs/metrics) and click on "edit" of the bigquery_valiation_metric you created in the previous post. Then add a label "job_name" of type String and field name:

```
protoPayload.metadata.jobChange.job.jobName
```

Do the same with a "query" label defined by field name:
```
protoPayload.metadata.jobChange.job.jobConfig.queryConfig.query
```

Update your metric and go to cloud monitoring. In order to get [alerts](https://console.cloud.google.com/monitoring/alerting) when BigQuery data quality errors are logged you need to [create a policy](https://console.cloud.google.com/monitoring/alerting/policies/create). Here's the [official guide/documentation](https://cloud.google.com/monitoring/alerts/using-alerting-ui#create-policy) but I will give some extra guidance for this particular use case.

1. Add condition
When adding a condition, add the following MQL in the query editor. Make sure to set a period long enough for you to act on the alert so that it doesn't "self heal" due to the aggregation period being to short. In the example below the period is set to 30 days, but probably something like 7 days is more reasonable.

```
fetch bigquery_project
| metric 'logging.googleapis.com/user/data-quality-error'
| group_by 30d, [value_error_aggregate: aggregate(value.error)]
| every 30d
| group_by [metric.query, metric.job_name],
    [value_error_aggregate_aggregate: aggregate(value_error_aggregate)]
| condition val() > 0 '1'

```

2. Who should be notified
Cloud monitoring offers a wide range of notification options. We'll use email in this case.

3. What are the steps to fix the issue




