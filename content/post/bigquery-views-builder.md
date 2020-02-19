---
title: "Automatic builds and version control of your BigQuery views"
date: 2020-02-19T7:50:33+01:00
draft: false
tags: ["DataHem","BigQuery", "Views", "Cloud Build"]
---

We (MatHem) has finally moved our BigQuery view definitions to GitHub and automized builds so that whenever someone in the data team modify/add a view definition and push/merge that to the master or develop branch it triggers a build of our views in our production/test environment respectively. Hence we get version control and always are in sync between the view definition and the views deployed in BigQuery.

Below are two ways to set it up and requires a github repo, cloud build and bigquery.

# Simple setup

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
CREATE OR REPLACE VIEW `<project_id>.commerce.view_cart_v1_reporting_latest`
OPTIONS(
description="the latest representation of each cart"
)
AS
SELECT * EXCEPT(row_number)
  FROM (
   SELECT
     *,
     ROW_NUMBER() OVER(PARTITION BY Id ORDER BY PartitionTimestamp DESC) row_number
   FROM(
    SELECT * FROM `<project_id>.commerce.cart_v1_reporting_timeseries`
      UNION ALL
    SELECT * FROM `<project_id>.commerce.cart_v1_staging` WHERE _PARTITIONDATE >= DATE_SUB(CURRENT_DATE(), INTERVAL 2 DAY)
   )
  ) 
  WHERE row_number = 1
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
The drawback is that everything is running in the same build and the build will succeed even if some views have errors.

# Advanced setup
A more advanced setup generates a build for each and every view, hence you can easily see what builds that fail and which that succeed. Let's do the necessary modifications.

The github repo should look like below where the folders under view will be the datasets in your BigQuery project:
```
/views/commerce/view_orders.sql
               /view_products.sql
      /payments/view_invoices.sql
cicd.sh
cloudbuild.yaml
cloudbuild.view.yaml
view.sh
```
Your view...sql files should look the same as in the simple setup. But the cicd.sh should now execute a build command rather than bq for every sql-file.

The cicd.sh
```bash
#!/bin/sh
#./cicd.sh mathem-ml-datahem-test views
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
  gcloud builds submit --async --config=cloudbuild.view.yaml --substitutions=_SQL_FILE="$file_entry" . 
done
```

The cloudbuild.yaml file looks the same as in simple setup. But we have an additional cloudbuild file, cloudbuild.view.yaml that execute the view.sh script with the projectId and sql-file as parameters.

cloudbuild.view.yaml
```yaml
# gcloud builds submit --config=cloudbuild.yaml .
steps:
- name: gcr.io/cloud-builders/gcloud
  entrypoint: 'bash'
  args: ['view.sh', '$PROJECT_ID', '${_SQL_FILE}']
timeout: 1200s
substitutions:
  _SQL_FILE: none
options:
    substitution_option: 'ALLOW_LOOSE'
```
And then we have the other extra file, the view.sh that execute the bq command that makes the views.

```bash
#!/bin/sh
project_id=$1
sql_file=$2
echo "$sql_file"
query="$(cat $sql_file)"
echo "${query//<project_id>/$project_id}"
bq query --batch --use_legacy_sql=false "${query//<project_id>/$project_id}"
```

# And last
Go to google cloud build, connect your github repository and create a trigger that listens to changes in your branch of choice, chose cloudbuild config and give it the path cloudbuild.yaml. Make sure you have enabled api:s for container registry and cloud build and that your projectnumber@cloudbuild.gserviceaccount.com has BigQuery Data Editor role in the project's IAM.

This is a small part of DataHem, our data platform at MatHem which we have open sourced to a large degree. Reach out on robert.sahlin[at]gmail.com or connect on [linkedin](https://www.linkedin.com/in/robertsahlin/) if you want to know more.
