---
title: "Flatten Firebase Properties and Parameters in Bigquery"
date: 2017-12-08T16:25:20+01:00
draft: false
tags: ["BigQuery", "Firebase", "UDF"]
id: "3"
---

At Google I/O May 2017, [Firebase](https://firebase.google.com/) announced [Google Analytics for Firebase](https://firebase.google.com/products/analytics/), a fantastic tool that automatically captures data on how people are using your iOS and Android app and lets you define your own custom app events. Like Google Analytics 360, it offers the ability to export raw data to Google BigQuery for custom analysis. There are a few posts on [Google Cloud Platform Blog](https://cloudplatform.googleblog.com/2016/09/using-BigQuery-and-Firebase-Analytics-to-understand-your-mobile-app.html) and [Firebase Blog](https://firebase.googleblog.com/2017/03/bigquery-tip-unnest-function.html) on how to query the Firebase dataset, but none of them giving much advise on how to analyze multiple properties and parameters at the same time. The reason why it is a bit tricky is that the dataset makes use of arrays of structs to flexibly accomodate user properties and event parameters. This is how the structure looks like:

![Firebase user properties schema in Google BigQuery](/images/firebase-user-properties-schema-bigquery.png)
![Firebase event parameters schema in Google BigQuery](/images/firebase-event-parameters-schema-bigquery.png)

I have previously written how you flatten your Google Analytics custom dimensions in Google BigQuery with the help of User Defined Functions and we can tackle the firebase challenge by applying the same logic. If don't have your own firebase dataset to play with, then [add the sample datset](https://bigquery.cloud.google.com/dataset/firebase-analytics-sample-data:android_dataset) 

This time we create two UDF:s, one for event parameters and one for user properties. They both accept a key and an array of structs mirroring the structure of properties and parameters and return a value struct.

{{< highlight sql >}}

#Standard-SQL

#UDF for event parameters
CREATE TEMP FUNCTION paramValueByKey(k STRING, params ARRAY<STRUCT<key STRING, value STRUCT<string_value STRING, int_value INT64, float_value FLOAT64, double_value FLOAT64 >>>) AS (
  (SELECT x.value FROM UNNEST(params) x WHERE x.key=k)
);

#UDF for user properties
CREATE TEMP FUNCTION propertyValueByKey(k STRING, properties ARRAY<STRUCT<key STRING, value STRUCT<value STRUCT<string_value STRING, int_value INT64, float_value FLOAT64, double_value FLOAT64>, set_timestamp_usec INT64, index INT64 > >>) AS (
  (SELECT x.value.value FROM UNNEST(properties) x WHERE x.key=k)
);

#Query the sample dataset, unnesting the events and turn 'api_version', 'round' and 'type_of_game' into columns 
SELECT 
  user_dim.user_id,
  event.name,
  propertyValueByKey('api_version', user_dim.user_properties).string_value AS api_version,
 paramValueByKey('round', event.params).int_value as round,
 paramValueByKey('type_of_game', event.params).string_value as type_of_game
FROM `firebase-analytics-sample-data.android_dataset.app_events_20160607`,
UNNEST(event_dim) as event
WHERE event.name = 'round_completed'
LIMIT 10;

{{< / highlight >}}

The result looks like this:
![Firebase flatten parameters and properties in Google BigQuery](/images/firebase-flatten-parameters-properties-bigquery.png)

Once again UDF:s to the rescue! Hopefully this helps some fellow firebase analysts out there. Please share with your network if you appreciate the example.