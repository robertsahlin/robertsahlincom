---
title: "Configure Firebase Analytics and Google Analytics app + web Bigquery export to EU region"
date: 2018-08-14T15:45:30+01:00
draft: false
tags: ["BigQuery", "Google Analytics"]
---

January 2019 when I tried to set up BigQuery export on our Firebase Analytics projects I found out that I couldn't chose region for the export and that it defaults to US. Since my employer is an European comapny I prefer to store the data in EU. This is the exact same issue that I had previously with GA360 BigQuery Export and I thought I would try to solve it in a similar manner (that has become part of the [the GA360 BigQuery Export documentation](https://support.google.com/analytics/answer/3416092?hl=en#step2.1) for how to geolocate your data in EU). The solution for Firebase Analytics and Google Analytics app plus web is the same and surprisingly easy.

When you activate/link BigQuery export in Firebase a dataset will be created in BigQuery when the first export is run. The dataset will created in your project and named as analytics_NNNNNNNNNN, you find the dataset name under Firebase > console > project settings > Integrations.

Hence, the solution is to:

1. Activate/link the BigQuery export in your Firebase project
2. Go to BigQuery and create a dataset under your Firebase project using your analytics_NNNNNNNNN name and EU as the region before the first export is executed. 
3. Wait and voila - your data starts pouring into your Bigquery dataset in the EU region!

The way the export works is that it first looks for a dataset with the ID and if it doesn't already exist then a dataset is created in the default region, i.e. US.

**Important!** This only applies to new exports. If you already have an existing export running, you need to take additional steps to not lose historical data when you change region to EU. Hence you have to:

1. Copy the existing dataset to a backup dataset 
2. Delete the existing export dataset
3. Follow the steps above
4. Copy the backup tables back into the newly created EU dataset.

I hope this prove to be useful for my fellow digital analysts in EU.
