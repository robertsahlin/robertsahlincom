---
title: "Get all unique Firebase Analytics events in BigQuery"
date: 2019-09-03T09:45:30+01:00
draft: false
tags: ["BigQuery", "Google Analytics", "Firebase Analytics"]
---

As I mentioned in my earlier [post about the drawbacks with the entity-attribute-value data model](https://robertsahlin.com/why-google-analytics-app---web-bigquery-export-rocks-and-sucks/) used in Firebase Analytics and Google Analytics app plus web, it is hard to know what events and associated attributes and data types are logged without proper documentation. Another way to get an overview is to actually query the table. Below you find an example of how to do it.

```SQL
SELECT 
event_name, 
ARRAY_AGG(struct(name, value)) as attribute FROM(
SELECT 
  event_name, 
  param.key as name, 
  CASE
    WHEN param.value.string_value is not null THEN "string"
    WHEN param.value.int_value is not null THEN "int"
    WHEN param.value.double_value is not null THEN "double"
    WHEN param.value.float_value is not null THEN "float"
  END as value
  FROM `<project>.<dataset>.events_20190902`, UNNEST(event_params) as param
  GROUP BY event_name, name, value)
GROUP BY event_name
ORDER BY event_name asc
```

That would return something like below (in JSON format)
```JSON
[
  {
    "event_name": "app_clear_data",
    "attribute": [
      {
        "name": "firebase_event_origin",
        "value": "string"
      }
    ]
  },
  {
    "event_name": "app_remove",
    "attribute": [
      {
        "name": "engaged_session_event",
        "value": "int"
      },
      {
        "name": "ga_session_id",
        "value": "int"
      },
      {
        "name": "firebase_event_origin",
        "value": "string"
      },
      {
        "name": "ga_session_number",
        "value": "int"
      }
    ]
  },
  {
    "event_name": "app_update",
    "attribute": [
      {
        "name": "engaged_session_event",
        "value": "int"
      },
      {
        "name": "previous_app_version",
        "value": "string"
      },
      {
        "name": "firebase_event_origin",
        "value": "string"
      },
      {
        "name": "ga_session_number",
        "value": "int"
      },
      {
        "name": "firebase_screen_id",
        "value": "int"
      },
      {
        "name": "ga_session_id",
        "value": "int"
      },
      {
        "name": "firebase_screen_class",
        "value": "string"
      }
    ]
  }
]
```

Hope this helps.
