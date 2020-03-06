---
title: "Unlimited persistent disk in google cloud shell"
date: 2020-03-05T7:50:33+01:00
draft: false
tags: ["Google Cloud Platform", "Cloud Shell", "VS Code Server"]
---

I use google cloud shell as my primary development environment. By doing that I can easily work from whatever computer I want as long as it has a browser and Internet connectivity. Cloud shell is free and comes with pretty much all the tools you need to develop services on Google Cloud Platform. But it comes with a huge limitation, it only provides 5 GB of persistent disk which won't last long if you work with software development. But I think I've found a solution, it works for me so far. It spells [Google Cloud Storage FUSE](https://cloud.google.com/storage/docs/gcs-fuse) and let you mount a GCS bucket as a folder to you linux instance. By doing that you get "unlimited" persistent disk (and backup) cheap as chips. And it is super simple to set up since gcsfuse is already installed in cloud shell!

![Unlimited persistent disk in cloud shell](/images/unlimited-persistent-disk.png)


```shell
# Replace [BUCKET_NAME], [USER] and [FOLDER_NAME] with yours 
gsutil mb gs://[BUCKET_NAME]/
mkdir /home/[USER]/[FOLDER_NAME]
chmod 777 /home/[USER]/[FOLDER_NAME]
gcsfuse -o nonempty -file-mode=777 -dir-mode=777 [BUCKET_NAME] /home/[USER]/[FOLDER_NAME]
```
If you don't want to mount gcs manually everytime you start the cloud shell, you can always add the gcsfuse command to a file named .customize_environment in your home directory (i.e. /home/[USER]/.customize_environment). I also use to start up a Visual Studio server which I use as IDE on port 8083 in web preview. Unfortunately the Theia editor won't pick up the mounted drive since Thei is started before mounting, and I honestly don't know how to restart it, I guess killing the process and starting it is one way but it doesn't feel right...

.customize_environment
```shell
#!/bin/sh
gcsfuse -o allow_other -o nonempty -file-mode=777 -dir-mode=777 --uid=1000 --debug_gcs [BUCKET_NAME] /home/[USER]/[FOLDER_NAME]
/home/[USER]/external/code-server2.1698-vsc1.41.1-linux-x86_64/code-server --auth none --port 8082
```

To install VS Code Server in the mounted folder
```shell
export VERSION_NAME=$(curl -s https://api.github.com/repos/cdr/code-server/releases/latest | jq -r .name)
export VERSION_TAG=$(curl -s https://api.github.com/repos/cdr/code-server/releases/latest | jq -r .tag_name)
wget -P /home/[USER]/external/ https://github.com/cdr/code-server/releases/download/$VERSION_TAG/code-server$VERSION_NAME-linux-x86_64.tar.gz
tar -xvzf /home/[USER]/external/code-server$VERSION_NAME-linux-x86_64.tar.gz --directory /home/[USER]/external/
rm -f /home/[USER]/external/code-server$VERSION_NAME-linux-x86_64.tar.gz
```
To start VS Code Server manually

```shell
external/code-server2.1698-vsc1.41.1-linux-x86_64/code-server --auth none --port 8082
```
