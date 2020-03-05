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
If you don't want to mount gcs manually everytime you start the cloud shell, you can always add the gcsfuse command to a file named .customize_environment in your home directory (i.e. /home/firstname_lastname/.customize_environment). I also use to start up a Visual Studio server which I use as IDE on port 8083 in web preview. Unfortunately the Theia editor won't pick up the mounted drive since Thei is started before mounting, and I honestly don't know how to restart it, I guess killing the process and starting it is one way but it doesn't feel right...

.customize_environment
```shell
#!/bin/sh
gcsfuse -o allow_other -o nonempty -file-mode=777 -dir-mode=777 --debug_gcs rs-gcsfuse /home/robert_sahlin/external
/home/robert_sahlin/code-server2.1523-vsc1.38.1-linux-x86_64/code-server --no-auth --port 8083
```