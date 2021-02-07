
This is part 2 in a series of data observability in GCP using only GCP native services (i.e. no third party tools or libraries). You find the first part [here](/validate-and-monitor-your-bigquery-data/).

If you followed part 1 you now have:

- scheduled data quality checks as SQL queries using BigQuery scheduled queries
- email alerts when mentioned queries report data quality errors
- logging of data quality errors
- a user defined data quality error metric
- a dashboard you can use to set alert thresholds in cloud monitoring

That's awesome. But now you want to take action on the errors in a structured way and even if e-mail is great it isn't the best place to get a sense of how frequent the error is and its status (raised, acknowledeged, resolved, etc.), especially if you are a data engineering team working on fixing alerts. So how can you accomplish that on GCP?

Enter [Error Reporting](https://cloud.google.com/error-reporting). This is a great service to analyze your errors in realtime, it also offers more advance notifications than what you get from BigQuery scheduled queries, you can even link errors to your issue tracker if you need to. The drawback, it currently only works for applications (cloud functions, cloud run, app engine, etc.) and not BigQuery.

So we just create a cloud function to report BigQuery data quality errors for us. But first we need to create a data quality [pubsub topic](https://console.cloud.google.com/cloudpubsub/topic/list) and name it to something like "data-quality-errors". 

Go back to the [legacy log viewer](https://console.cloud.google.com/logs/viewer) and click on the button "create sink" (top menu). Create a sink according to the picture below and use a log inclusion filter like this one.

```
resource.type="bigquery_project"
resource.labels.location="EU"
severity>=ERROR
```

![picture](/images/logs-routing-sink.png)

Now you should be able to see you newly created sink in the [logs router](https://console.cloud.google.com/logs/router).

