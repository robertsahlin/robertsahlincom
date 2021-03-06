---
title: "Validate and monitor your BigQuery data"
date: 2021-02-06T13:50:33+01:00
draft: false
tags: ["BigQuery", "Validation", "Observability"]
---

Data observability has gained huge momentum and data quality is essential for any kind of analytical system no matter it is plain old reporting or advanced machine learning. I've seen reports that states that data engineers spend more than 30% of their time manually chasing data quality issues! That is not only cost in term of precious resources's time but also missed opportunities or even worse - loss in trust of your data and your data team.

So, how to validate your data in BigQuery? There are a number of approaches (rules, statistical tests, ML) to unit test your data or identifying anomalies:

- Airflow (Cloud Composer) BigQuery operators
- BigQuery scheduled queries + cloud monitoring
- Open source (Great Expectations, Tensorflow Data Validation, Deequ, Apache Griffin, etc.)
- Third party services (Monte Carlo, Validio, etc.)

At MatHem we currently use Cloud Composer's (Airflow) BigQuery operators to run different SQL queries to validate if data meet expected results. But if you don't use cloud composer and still want to validate your data quality there is an alternative setup I want to present - BigQuery scheduled queries + cloud monitoring.

First, did you know you can throw errors in BigQuery?

```sql
SELECT ERROR("this is an error")
```

That means you can set a condition to throw an error in BigQuery, like this.

```sql
SELECT IF(true, ERROR("this is an error"), "this is ok")
```

As an example, let's say you have a query for which you expect 6 more or more ids as a result. Then you can throw an error if it doesn't meet that condition.

```sql
SELECT 
    IF(COUNT(DISTINCT id) < 6, ERROR("validation error: ids are < 6"), "ok") as validation
FROM(
    SELECT * FROM UNNEST(GENERATE_ARRAY(1, 5)) AS id
)
```

Now you can create [scheduled query in BigQuery](https://cloud.google.com/bigquery/docs/scheduling-queries) using a validation query like the one above (one that throws an error if it doesn't meet expected result). If you set the option to send email notifications, then you will receive an email when your validation job fails (throws an error). The email contains a brief explanation and a link to the scheduled job. Easy as pie!
![BigQuery scheduled query notification email](/images/bigquery-scheduled-query-notification.png)

But it doesn't stop there. Getting alerts in your email is great, but you probably want a more sophisticated way to monitor your validation. Enter Cloud Logging and Cloud Monitoring.

First, open the [legacy logs viewer](https://console.cloud.google.com/logs/viewer). And filter entries with something like below and you should see the entry representing your failed job.

```
resource.type="bigquery_project"
resource.labels.location="EU"
severity>=ERROR
```
![logs viewer](/images/logs-viewer.png)

Now click the Create Metric button in the top of the screen and a side bar will open to define your log based metric. Give it a name and description, but also custom labels such as error_message defined with field name "protoPayload.status.message". 

![user defined metrics](/images/user-defined-metric.png)

When saved you will find it under "User defined metrics" in the [Logs based metrics menu](https://console.cloud.google.com/logs/metrics). Click the symbol with three bullets to the right for your new metric and chose create alert, that will take you to [cloud monitoring > alerts](https://console.cloud.google.com/monitoring/alerting). Give your policy a name and set treshold configuration and save. Now you have a metric dashboard and alerts for your validation errors.

![alerting plicy](/images/alerting-policy.png)
