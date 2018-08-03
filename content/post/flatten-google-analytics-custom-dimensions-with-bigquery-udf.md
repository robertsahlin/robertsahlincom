---
title: "Flatten Google Analytics Custom Dimensions with a BigQuery UDF"
date: 2017-10-30T16:35:11+01:00
draft: false
tags: ["BigQuery", "Google Analytics", "UDF"]
id: "4"
---

**Updated 2018-04-23 with a fourth alternative - Unnest**

Are you one of the lucky digital analysts that have a google analytics premium account? Then you know you can export your data to Google BigQuery and analyze it in an adhoc and explorative manner using SQL. One frequent use case for BigQuery is to analyze many custom dimensions at the same time. But there is a challenge in how to do that in BigQuery since it follows a nested/repeated pattern.

Let's use the [public google analytics sample "LondonCycleHelmet"](https://support.google.com/analytics/answer/3416091?hl=en) and say you want extract the custom dimensions 1-3 on hit level. I don't know what the dimensions represent so I've made up that they are productCategory, loyaltyClass and existingCustomer. So you want to see the named custom dimensions together with id:s for visitor, session and hit. Since custom dimensions are nested/repeated you need to use one of the following tricks to turn multiple rows into multiple columns:

1. Max/Case
2. Custom Javascript UDF
3. Custom SQL UDF
4. Unnest (recommended)

This data transformation of flattening a table is also called "pivot", but BigQuery doesn't support that natively, yet. I will go through each of the tricks below.

Max/Case
---
If the custom dimension is assigned on hit level then you group them on the hit level and in order to do that you need to have a unique hit ID and in this case it is the combination of fullvisitorid, visitid and hitnumber in the Google Analytics dataset.

{{< highlight sql >}}
#Standard-SQL
SELECT 
  fullvisitorid,
  visitid,
  hit.hitnumber,
  max(case when customdimension.index = 1 then customdimension.value end) productCategory,
  max(case when customdimension.index = 2 then customdimension.value end) loyaltyClass,
  max(case when customdimension.index = 3 then customdimension.value end) existingCustomer
FROM `google.com:analytics-bigquery.LondonCycleHelmet.ga_sessions_20130910`,
UNNEST(hits) as hit,
UNNEST(hit.customdimensions) as customdimension
GROUP BY fullvisitorid, visitid, hit.hitnumber
LIMIT 100
{{< / highlight >}}


Custom Javascript UDF
---
An alternative approach using a User Defined Function (UDF) solving the same use case. Since we call the UDF with the custom dimensions as an array, we don't have to unnest the custom dimensions in the SQL, and the syntax becomes cleaner. 

{{< highlight sql >}}
#Standard-SQL
CREATE TEMPORARY FUNCTION customDimensionByIndex(index INT64, arr ARRAY<STRUCT<index INT64, value STRING>>)
RETURNS STRING
LANGUAGE js AS """
  for (var j = 0; j < arr.length; j++){
    if(arr[j].index == index){
      return arr[j].value;
    }
  }""";

SELECT 
  fullvisitorid,
  visitid,
  hit.hitnumber,
  customDimensionByIndex(1, hit.customDimensions) as productCategory,
  customDimensionByIndex(2, hit.customDimensions) as loyaltyClass,
  customDimensionByIndex(3, hit.customDimensions) as existingCustomer
FROM `google.com:analytics-bigquery.LondonCycleHelmet.ga_sessions_20130910`,
UNNEST(hits) as hit
LIMIT 100
{{< / highlight >}}

The UDF may cause a performance hit, but it is a choice between simplicity in writing SQL and the performance running it. The UDF is generic enough for you to apply it on all levels of custom dimension, i.e. session, hit or product.

Custom SQL UDF
---
The third option is to use a SQL UDF which should improve performance. Felipe Hoffa (Developer Advocate at Google) was kind enough to port the Javascript UDF to a [SQL UDF](https://stackoverflow.com/a/44301282/132438).

{{< highlight sql >}}
#standardSQL
CREATE TEMP FUNCTION customDimensionByIndex(indx INT64, arr ARRAY<STRUCT<index INT64, value STRING>>) AS (
  (SELECT x.value FROM UNNEST(arr) x WHERE indx=x.index)
);

SELECT 
  fullvisitorid,
  visitid,
  hit.hitnumber,
  customDimensionByIndex(1, hit.customDimensions) as productCategory,
  customDimensionByIndex(2, hit.customDimensions) as loyaltyClass,
  customDimensionByIndex(3, hit.customDimensions) as existingCustomer
FROM `google.com:analytics-bigquery.LondonCycleHelmet.ga_sessions_20130910`,
UNNEST(hits) as hit
LIMIT 100
{{< / highlight >}}

UNNEST
---
There is one limitation with Custom SQL UDF, you can't use them in views. You can use javascript UDF:s in views, but I thought I would show a fourth alternative to flatten your custom dimensions. 
My intention was to use the [newly released Google Analytics dataset](https://support.google.com/analytics/answer/7586738?hl=en) from the Google Merchandise Store, something I've been requesting for some time. However, the dataset doesn't seem to have custom dimensions on hit level at the time of writing.

{{< tweet 954821355194716160 >}}

{{< highlight sql >}}
#UNNEST
SELECT
fullvisitorId,
visitid,
hit.hitnumber,
(SELECT x.value FROM UNNEST(hit.customDimensions) x WHERE x.index = 1) as productCategory,
(SELECT x.value FROM UNNEST(hit.customDimensions) x WHERE x.index = 2) as loyaltyClass,
(SELECT x.value FROM UNNEST(hit.customDimensions) x WHERE x.index = 3) as existingCustomer
FROM `google.com:analytics-bigquery.LondonCycleHelmet.ga_sessions_20130910`,
UNNEST(hits) as hit
LIMIT 100
{{< / highlight >}}