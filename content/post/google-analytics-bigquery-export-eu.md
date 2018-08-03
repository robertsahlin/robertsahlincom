---
title: "Configure Google Analytics Bigquery export to EU region"
date: 2017-10-30T15:45:30+01:00
draft: false
tags: ["BigQuery", "Google Analytics"]
cd1: "test"
id: "5"
---

**(Updated 2018-01-23):** Google has now included my input in [the GA360 BigQuery Export documentation](https://support.google.com/analytics/answer/3416092?hl=en#step2.1) for how to geolocate your data in EU.

***

April 2017 when I tried to set up BigQuery export on one of our Google Analytics premium views I found out that I couldn't chose region for the export and that it defaults to US. Since my employer is an European comapny I prefer to store the data in EU.
Google wasn't aware of the problem themselves and filed a feature request to the Cloud Platform engineering team. However, I couldn't wait for that and managed to come up with a solution myself, it was surprisingly easy:

When I activate bigquery export a dataset is created in bigquery. The dataset is named with a number and I noticed that this ID is the same as the ID for the view I want to export.

Hence, the solution is to:

1. Create a dataset in Bigquery with the view ID as name and EU as the region. 
2. Setup the export in Google Analytics
3. Wait and voila - your GA data starts pouring into your Bigquery dataset in the EU region!

I think the way the export works is that it first looks for a dataset with the view ID and if it doesn't already exist then a dataset is created in the default region, i.e. US.

**Important!** This only applies to new exports. If you already have an existing export running, you need to take additional steps to not lose historical data when you change region to EU. The reason is that Google Analytics export only backfill once (weird limitation, I’ve requested Google to fix it). Hence you have to:

1. Copy the existing dataset to a backup dataset 
2. Delete the existing export dataset
3. Follow the steps above
4. Copy the backup tables back into the newly created EU dataset.

I hope this prove to be useful for my fellow digital analysts in EU.

![Google analytics view to be used in BigQuery Export](/images/google-analytics-view-bq-export.png)

![Create dataset in BigQuery](/images/bigquery-create-dataset-region.png)