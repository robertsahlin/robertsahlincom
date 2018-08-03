---
title: "Serverless Static Blog: Hugo + Github + Cloud Build + Firebase"
date: 2018-08-03T10:33:33+01:00
draft: true
tags: ["Hugo", "Github", "Google Cloud Build", "Firebase"]
---

One of the announcements I liked the most from Google Cloud Next 18 was Google Cloud Build (former Google Container Builder). I've been missing easy and lightweight CI/CD for GCP, especially with focus on serverless. I thought I would give it a try before setting it up for CI/CD in the open source data/analytics/ML solution I'm working on - [DataHem](https://github.com/mhlabs/datahem). Hence, I figured I use it to power this blog.

The project directory of this blog is hosted on [Github](https://github.com/) and use the awesome generator [Hugo](https://gohugo.io/) to generate the static site that is hosted on [Firebase](https://firebase.google.com/). What I wanted to accomplish is to set up a build that use a push to Github to trigger a build that reads in the project directory from Github, run Hugo to generate HTML-files and push those to Firebase hosting. Actually, this post is written using Githubs web client and when committed trigger a build as described above.

#### 1. Hugo
First you need to generate the project directory with hugo. Get started by following the [quick start instructions](https://gohugo.io/getting-started/quick-start/).

#### 2. Firebase
[Follow the instructions to set up and deploy to Firebase](https://gohugo.io/hosting-and-deployment/hosting-on-firebase/)

#### 3. Git
Initialize a local git in your project directory root (if not already done to download a theme). Make sure you ignore the /public folder in .gitignore (or remember to add a step in cloud build to remove that folder before running the hugo command step). Also, delete the .git in the themes folder (if there is any) to avoid issues when cloud build try to run hugo. Create a github repo and add as remote.

#### 4. Google Cloud Build
git community builders -> edit firebase. submit to container registry
create KMS key
cloudbuild.yaml in project directory root
create a trigger
push project directory to github repo
