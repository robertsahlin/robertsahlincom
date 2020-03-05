---
title: "Unlimited persistent disk in google cloud shell"
date: 2020-03-05T7:50:33+01:00
draft: false
tags: ["Google Cloud Platform", "Cloud Shell"]
---

I use google cloud shell as my primary development environment. By doing that I can easily work from whatever computer I want as long as it has a browser and Internet connectivity. Cloud shell is free and comes with pretty much all the tools you need to develop services on Google Cloud Platform. But it comes with a huge limitation, it only provides 5 GB of persistent disk which won't last long if you work with software development. But I think I've found a solution, it works for me so far. It spells [Google Cloud Storage FUSE](https://cloud.google.com/storage/docs/gcs-fuse) and let you mount a GCS bucket as a folder to you linux instance. By doing that you get "unlimited" persistent disk (and backup) cheap as chips. And it is super simple to set up since gcsfuse is already installed in cloud shell!

```shell
gsutil mb gs://[BUCKET_NAME]/
mkdir /home/firstname_lastname/[FOLDER_NAME]
chmod 777 /home/firstname_lastname/[FOLDER_NAME]
gcsfuse -o nonempty -file-mode=777 -dir-mode=777 [BUCKET_NAME] /home/firstname_lastname/[FOLDER_NAME]
```
and that's it!
