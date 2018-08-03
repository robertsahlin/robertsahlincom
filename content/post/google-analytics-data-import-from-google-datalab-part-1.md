---
title: "Google Analytics Data Import From Google Datalab Part 1"
date: 2018-01-11T09:04:27+01:00
draft: true
tags: ["Datalab", "BigQuery", "Google Analytics"]
id: "7"
---

I've been doing some analytics with [Google Datalab](https://cloud.google.com/datalab/) (easy to use interactive tool for data exploration, analysis, visualization and machine learning) the last year. My focus has been explorative analytics of the Google Analytics data that we export to BigQuery. But what is data science if we don't incorporate the data back into the real world to enrich the user experience? Hence I thought I would show how you to import your data insights back into Google Analytics.

First we need to set up datalab in the cloud accordingly, I'll assume you already have a project set up in Google Cloud Platform and enabled required API:s/services. There is a [video giving a good introduction](https://www.youtube.com/watch?v=Eu57QKNHaiY&feature=em-subs_digest-vrecs), but missing steps such as datalab for teams and setting up scopes for pushing data back to Google Analytics. Also, I do all commands from the [Google Cloud Shell](https://cloud.google.com/shell/docs/features) that already has the necessary components installed and also provides a [built-in code editor - Orion](https://cloud.google.com/shell/docs/features#code_editor). If you run gcloud from command line locally, you need to install the datalab component first.

Since I'm mostly working in teams using Google datalab I prefer using a common service account to limit access to services and set scopes. Hence I [create a service account](https://cloud.google.com/iam/docs/creating-managing-service-accounts) and give it the required roles. Then, I [add each user to the project](https://cloud.google.com/iam/docs/granting-changing-revoking-access#granting_access_to_team_members) and give them [IAM roles for the service account that we will attach to the user's Cloud Datalab instance](https://cloud.google.com/datalab/docs/how-to/datalab-team#project_owner_creates_instances_for_other_team_members)

{{< highlight Bash>}}
#Create service account
gcloud iam service-accounts create myserviceaccount \
    --display-name "my service account"

#Give required roles to service account
gcloud projects add-iam-policy-binding myproject \
    --member serviceAccount:myserviceaccount@myproject.iam.gserviceaccount.com \
    --role roles/bigquery.user

gcloud projects add-iam-policy-binding myproject \
    --member serviceAccount:myserviceaccount@myproject.iam.gserviceaccount.com \
    --role roles/bigquery.jobUser

gcloud projects add-iam-policy-binding myproject \
    --member serviceAccount:myserviceaccount@myproject.iam.gserviceaccount.com \
    --role roles/bigquery.dataViewer

#Add user and give required roles
gcloud projects add-iam-policy-binding myproject \
    --member user:myname@mycompany.com \
    --role roles/iam.serviceAccountUser

gcloud projects add-iam-policy-binding myproject \
    --member user:myname@mycompany.com \
    --role roles/compute.instanceAdmin.v1
{{< / highlight >}}

Thereafter, [create a Google datalab instance](https://cloud.google.com/datalab/docs/reference/command-line/create#usage). Here I create an instance called datalab-myname for user myname@mycompany.com using the project myproject.

{{< highlight Bash>}}
datalab create "datalab-myname" \
    --disk-size-gb "20" \
    --for-user myname@mycompany.com \
    --project "myproject" \
    --zone "europe-west1-b"
{{< / highlight >}}

But that is not enough, we need to change scopes as well since the default scopes don't include analytics. In order to change scopes you fist need to stop the instance:

{{< highlight Bash>}}
gcloud beta compute instances stop "datalab-myname" \
    --zone="europe-west1-b"
{{< / highlight >}}

Then we set scopes to also include the google analytics service:

{{< highlight Bash>}}
gcloud beta compute instances set-scopes datalab-myname \
    --service-account "myserviceaccount@myproject.iam.gserviceaccount.com" \
    --scopes "https://www.googleapis.com/auth/cloud-platform,\
https://www.googleapis.com/auth/analytics.edit,\
https://www.googleapis.com/auth/analytics,\
https://www.googleapis.com/auth/bigquery,\
https://www.googleapis.com/auth/devstorage.read_write,
https://www.googleapis.com/auth/compute"
{{< / highlight >}}

Then restart the instance

{{< highlight Bash>}}
gcloud beta compute instances start "datalab-myname" \
    --zone="europe-west1-b"
{{< / highlight >}}

Then give the service account (myserviceaccount@myproject.iam.gserviceaccount.com) edit priviliges for each account/property in Google Analytics admin settings and you're done!

Now the user (myname@mycompany.com) can connect to Google datalab by going to the cloud shell and type:

{{< highlight Bash>}}
datalab connect "datalab-myname" \
    --zone "europe-west1-b"
{{< / highlight >}}

Stop the instance when you are finished to avoid being charged more than what is needed:

{{< highlight Bash>}}
datalab stop "datalab-myname"
{{< / highlight >}}

In the next posts we'll go through the code to query BigQuery and prepare the results for Google Analytics data import and finally upload the data and save the notebook to Google Source repositories using git.