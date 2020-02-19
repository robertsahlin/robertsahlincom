---
title: "Automatic builds and version control of your BigQuery schemas"
date: 2020-02-19T7:50:33+01:00
draft: false
tags: ["DataHem","BigQuery", "Views", "Cloud Build"]
---

We (MatHem) has finally moved our BigQuery view definitions to GitHub and automized builds so that whenever someone in the data team modify/add a view definition and push/merge that to the master or develop branch it triggers a build of our views in our production/test environment respectively. Hence we get version control and always are in sync between the view definition and the views deployed in BigQuery.

It's a very simple setup and requires a github repo, cloud build and bigquery.

The github repo should look like:
```
/views
cicd.sh
cloudbuild.yaml
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
- id: 'update views'
  name: 'gcr.io/${PROJECT_ID}/bq:${_BQ_TAG}'
  entrypoint: 'bash'
  args: ['cicd.sh', '$PROJECT_ID', '${_VIEWS_DIR}']
timeout: 1200s
substitutions:
  _VIEWS_DIR: views
  _BQ_TAG: latest
```

You also need to add the [bq community builder](https://github.com/GoogleCloudPlatform/cloud-builders-community/tree/master/bq) to your container registry before you can run the build.

```bash
#Clone the cloud-builders-community repo:
$ git clone https://github.com/GoogleCloudPlatform/cloud-builders-community

Go to the directory that has the source code for the bq Docker image:
$ cd cloud-builders-community/bq

Build the Docker image:
$ gcloud builds submit --config cloudbuild.yaml .
```

Go to google cloud build, connect your github repository and create a trigger that listens to changes in your branch of choice, chose cloudbuild config and give it the path cloudbuild.yaml. Make sure you have enabled api:s for container registry and cloud build and that your projectnumber@cloudbuild.gserviceaccount.com has BigQuery Data Editor role in the project's IAM.

This is a small part of DataHem, our data platform at MatHem which we have open sourced to a large degree. Reach out on robert.sahlin[at]gmail.com or connect on [linkedin](https://www.linkedin.com/in/robertsahlin/) if you want to know more.
