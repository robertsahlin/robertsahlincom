---
title: "Automatic builds and version control of your BigQuery views"
date: 2020-02-19T7:50:33+01:00
draft: false
tags: ["DataHem","BigQuery", "Views", "Cloud Build"]
---

We (MatHem) has finally moved our BigQuery view definitions to GitHub and automized builds so that whenever someone in the data team modify/add a view definition and push/merge that to the master or develop branch it triggers a build of our views in our production/test environment respectively. Hence we get version control and always are in sync between the view definition and the views deployed in BigQuery.

It's a very simple setup and requires a github repo, cloud build and bigquery.

The github repo should look like below where the folders under view will be the datasets in your BigQuery project:
```
/views/commerce/view_orders.sql
               /view_products.sql
      /payments/view_invoices.sql
cicd.sh
cloudbuild.yaml
```
Your view...sql files should follow the DDL syntax for views but replace the project with <project_id> if you want it dynamically.

Example:
```sql
CREATE OR REPLACE VIEW `<project_id>.audit.view_bq_hourly_cost`
OPTIONS(
description="view BigQuery cost per hour"
)
AS SELECT
  FORMAT_TIMESTAMP('%F', protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.job.jobStatistics.endTime, 'Europe/Stockholm') as day,
  FORMAT_TIMESTAMP('%H:00', protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.job.jobStatistics.endTime, 'Europe/Stockholm') as hour,
  FORMAT('%9.2f',50.0 * (SUM(protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.job.jobStatistics.totalBilledBytes)/POWER(2, 40))) as Estimated_SEK_Cost
FROM
  `<project_id>.audit.cloudaudit_googleapis_com_data_access_*`
WHERE
  protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.eventName = 'query_job_completed'
GROUP BY day, hour
ORDER BY day, hour DESC
```

The cicd.sh
```bash
#!/bin/sh
#./cicd.sh my-project-id views EU
project_id=$1
views_dir=$2
location=${3:-EU}  

bq_safe_mk() {
    dataset=$1
    exists=$(bq ls -d | grep -w $dataset)
    if [ -n "$exists" ]; then
       echo "Not creating $dataset since it already exists"
    else
       echo "Creating dataset $project_id:$dataset with location: $location"
       bq --location=$location mk $project_id:$dataset
    fi
}

for dir_entry in $(find ./$views_dir -mindepth 1 -maxdepth 1 -type d -printf '%f\n')
do
  echo "$dir_entry"
  bq_safe_mk $dir_entry
done

for file_entry in $(find ./$views_dir -type f -follow -print)
do
  echo "$file_entry"
  query="$(cat $file_entry)"
  echo "${query//<project_id>/$project_id}"
  bq --nosync query --batch --use_legacy_sql=false "${query//<project_id>/$project_id}"
done
```

cloudbuild.yaml
```yaml
# gcloud builds submit --config=cloudbuild.yaml .
steps:
- name: gcr.io/cloud-builders/gcloud
  entrypoint: 'bash'
  args: ['cicd.sh', '$PROJECT_ID', '${_VIEWS_DIR}']
timeout: 1200s
substitutions:
  _VIEWS_DIR: views
```

Go to google cloud build, connect your github repository and create a trigger that listens to changes in your branch of choice, chose cloudbuild config and give it the path cloudbuild.yaml. Make sure you have enabled api:s for container registry and cloud build and that your projectnumber@cloudbuild.gserviceaccount.com has BigQuery Data Editor role in the project's IAM.

This is a small part of DataHem, our data platform at MatHem which we have open sourced to a large degree. Reach out on robert.sahlin[at]gmail.com or connect on [linkedin](https://www.linkedin.com/in/robertsahlin/) if you want to know more.
