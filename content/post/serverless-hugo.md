---
title: "Serverless Static Blog powered by Hugo, Github, Cloud Build and Firebase"
date: 2018-08-03T16:33:33+01:00
draft: false
tags: ["Hugo", "Github", "Google Cloud Build", "Firebase"]
---

One of the announcements I liked the most from Google Cloud Next 18 was [Google Cloud Build](https://cloud.google.com/cloud-build/) (former Google Container Builder). I've been missing easy and lightweight CI/CD for GCP, especially with focus on serverless. I thought I would give it a try before setting it up for CI/CD in the open source data/analytics/ML solution I'm working on - [DataHem](https://github.com/mhlabs/datahem). Hence, I figured I use it to power this blog.

The project directory of this blog is hosted on [Github](https://github.com/) and use the awesome generator [Hugo](https://gohugo.io/) to generate the static site that is hosted on [Firebase](https://firebase.google.com/). What I wanted to accomplish is to set up a build that use a push to Github to trigger a build that reads in the project directory from Github, run Hugo to generate HTML-files and push those to Firebase hosting. Actually, this post is written using Githubs web client and when committed trigger a build as described above.

#### 1. Hugo
First you need to generate the project directory with hugo. Get started by following the [quick start instructions](https://gohugo.io/getting-started/quick-start/).

#### 2. Firebase
[Follow the instructions to set up and deploy to Firebase](https://gohugo.io/hosting-and-deployment/hosting-on-firebase/)

#### 3. Git
Initialize a local git in your project directory root (if not already done). Make sure you ignore the /public folder in .gitignore (or remember to add a step in cloud build to remove that folder before running the hugo command step). Also, delete the .git in the themes folder (if there is any) to avoid issues when cloud build try to run hugo. Create a github repo and add as remote in your local git.

#### 4. Google Cloud Build
Google Cloud Build has some [builders maintained by Google](https://github.com/GoogleCloudPlatform/cloud-builders) and [some builders are open-source and contributed by the community](https://github.com/GoogleCloudPlatform/cloud-builders-community). I use two community maintained builders, hugo and firebase. 

Create a builders directory separate from your project directory, init git and clone the cloud-builders-community repo. 

##### 4.1 Hugo Builder Image
Go to the directory that has the source code for the hugo Docker image and build the image (make sure you have gcloud installed):
```bash
$ cd cloud-builders-community/hugo
$ gcloud builds submit --config cloudbuild.yaml .
```
##### 4.2 Firebase Builder Image
There seem to be a bug in the firebase builder, so before you go on building the firebase image, remove the the line with secretEnv and change privilages on the firebase.bash to be executable.

```yaml
#cloudbuild.yaml should look like this
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/firebase', '.']
images:
- 'gcr.io/$PROJECT_ID/firebase'
```

```bash
$ cd cloud-builders-community/hugo
chmod +x firebase.bash
$ gcloud builds submit --config cloudbuild.yaml .
```

##### 4.3 cloudbuild.yaml
Now it's time to create a build configuration file. Go to you project directory root and create a file named cloudbuild.yaml. The file should look like
```yaml
steps:
- name: 'gcr.io/$PROJECT_ID/hugo'
  args: ['-t', 'casper','-v']
- name: 'gcr.io/$PROJECT_ID/firebase'
  args: ['deploy']
  secretEnv: ['FIREBASE_TOKEN']
secrets:
- kmsKeyName: 'projects/[PROJECT_ID]/locations/global/keyRings/cloudbuilder/cryptoKeys/firebase-token'
  secretEnv:
    FIREBASE_TOKEN: '<YOUR_ENCRYPTED_TOKEN>'
```

Replace [PROJECT_ID] and <YOUR_ENCRYPTED_TOKEN> by following the instructions for how to create a Google KMS key in [firebase builder readme.md](https://github.com/GoogleCloudPlatform/cloud-builders-community/blob/master/firebase/README.md)

##### 4.4 Build trigger
Create a build trigger to automate builds by following the [guide on Google Cloud Builder](https://cloud.google.com/cloud-build/docs/running-builds/automate-builds), i.e. select Github, authenticate select the remote repo and that's about it.

##### 4.5 Push to trigger build
In your local project directory git, add files, commit and make your first push to trigger a build.
```bash
$ git add .
$ git commit -m "my first commit"
$ git push -u origin master
```
Your github repo's root directory should look like:
![file structure on github](/images/github_repo_directory_structure.png)

Now you should see a build in the [console](https://console.cloud.google.com/cloud-build/builds) and if it's green then you should also see it in the firebase deployment history. Remember to set up a custom domain in firebase hosting if you have one.

Happy serverless blogging!
