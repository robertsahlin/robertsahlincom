---
title: "Google Analytics Custom Dimension Alias in Bigquery"
date: 2017-12-07T16:30:54+01:00
draft: false
tags: ["BigQuery", "Google Analytics", "User Defined Functions"]
id: "6"
---

Second to being able to export your Google Analytics data to Google BigQuery, the feature I value the most with the premium version of GA is that you are not limited to 20 custom dimensions but have 200 to play with! However, if you have many custom dimensions, it quickly becomes hard to remember what dimension each index represents, the value isn't always selfdescribing. Hence being able to give the custom dimension a more descriptive identifier than an index could be useful.

This is one way of doing it without having to explicitly state the alias in your query, but rather have a dimension table to make lookups within the custom dimension array, so that all analysts use the same terminology and mirror that of what you may use in Google Analytics.

{{< highlight sql >}}
#Standard-SQL

#User Defined Function
CREATE TEMP FUNCTION customDimensionLookup(
  fact_arr ARRAY<STRUCT<index INT64, value STRING>>,
  dim_arr ARRAY<STRUCT<index INT64, value STRING>>) 
  RETURNS ARRAY<STRUCT<name STRING, value STRING>> AS (
    (SELECT ARRAY_AGG(STRUCT( y.value as name, x.value as value)) as arr FROM UNNEST(fact_arr) x LEFT JOIN UNNEST(dim_arr) y ON x.index = y.index)
);

#Replace with a real lookup table, this example inline table is to show the contents
WITH dim_table AS(
SELECT 1 AS index, 'product name' as value
UNION ALL
SELECT 2 AS index, 'membership' as value
UNION ALL
SELECT 3 AS index, 'newsletter subscriber' as value
),
#Transform rows in lookup table to an array since tables can't referenced in a UDF
dim_array AS (
  SELECT
     ARRAY_AGG(STRUCT(
       index as index,
       value AS value)) as arr
   FROM
   dim_table
)
#Query the example GA dataset to show difference of original custom dimension array and the new with alias
SELECT
  fullvisitorid,
  customDimensions,
  customDimensionLookup(customDimensions, arr) as customDimensionsAlias
  FROM `google.com:analytics-bigquery.LondonCycleHelmet.ga_sessions_20130910`, dim_array
  WHERE ARRAY_LENGTH(customDimensions) > 0
  LIMIT 10
{{< / highlight >}}

User defined functions are really powerful way to transform your data to a more suitable structure/format and I urge to develop your own. The result looks like this:
![Custom Dimension alias in in BigQuery](/images/bigquery-custom-dimension-alias.png)