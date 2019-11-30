---
title: "Schema evolution in streaming Dataflow jobs and BigQuery tables, part 2"
date: 2019-11-30T7:50:33+01:00
draft: true
tags: ["DataHem", "Protobuf", "Schema", "Apache Beam", "BigQuery", "Dataflow"]
---

In the [previous post](https://robertsahlin.com/schema-evolution-in-streaming-dataflow-jobs-and-bigquery-tables-part-2/), I covered how we create or patch BigQuery tables without interrupting the real-time ingestion. This post will focus on how we update the dataflow (Apache Beam) job without interrupting the real-time ingestion.

![DataHem Architecture](/images/datahem_architecture_v2.png)

# 3 Dataflow
Google Cloud Dataflow is a fully managed service for executing Apache Beam pipelines within the Google Cloud Platform ecosystem. Apache Beam is an open source unified programming model to define and execute data processing pipelines, including ETL, batch and stream (continuous) processing. I've been developing ETL-jobs and pipelines in Hadoop (Hive, Pig, MapReduce) and Spark and discovered Apache Beam 2 years ago and never looked back, Apache Beam is awesome!

## 3.1 Generic pipelines
To avoid developing code for multiple data object pipelines that share 99% percent of the code (only the generated protobuf message class differs) we decided to develop a generic pipeline that can interpret messages dynamically based on a schema that is read at runtime rather than at compilation. 

The pipeline accepts parameters such as what pubsub subscription to read messages from, where to find the file descriptor (our schema registry is a file descriptor in cloud storage), the full name of the message schema and what BigQuery table to write to. The pipeline is run in streaming mode and use small instances since it doesn't do any aggregations. We set up one pipeline for each data object.

The generic pipeline parse JSON against the specified protobuf schema and alert if fields are missing (but still processing the record). The protobuf message is the transformed into a tablerow according to the message and field annotations in the schema. 

The entities are written to a staging table in BigQuery that is partitioned on ingestion time, hence we can run backfills to the same pipeline without facing limitations in partitioning records older than 30 days. We have two very similar pipelines, one for reading entities from dynamoDB streams, publish JSON with NewImage and OldImage of each entity, and one for entities with only one version (Image) of each entity.

Docker images...

Dataflow jobs can easily be updated as long as the pipeline DAG doesn't change, hence we can update streaming jobs when the schema is updated, without interuptions. 


## 3.2 DataHem.Patcher
DataHem.Patcher is a Java application that I developed to patch BigQuery table schemas from a descriptor file (see part 1 in this series of posts) stored in cloud storage. The application is invoked with a configuration as below:

```json
{
        "patches":[
            {
                "fileDescriptorBucket":"${_PROJECT_ID}-descriptor",
                "fileDescriptorName":"${_FILE_DESCRIPTOR_NAME}",
                "descriptorFullName":"${_DESCRIPTOR_FULL_NAME}",
                "policyTagPattern":"${_POLICY_TAG_PATTERN}",
                "tables":[
                    {
                        "tableReference":{
                            "projectId":"${_PROJECT_ID}",
                            "datasetId":"${_DATASET}",
                            "tableId":"${_TABLE}_staging"
                        },
                        "createDisposition":"CREATE_IF_NEEDED",
                        "timePartitioning":{
                            "field":null,
                            "requirePartitionFilter": true
                        },
                        "clustering":{
                            "fields":[]
                        }
                    }
         ]
 }
```
The application loops through one or multiple patches and tables and support settings such as create disposition, time partitioning and clustering. The actual fields are described in the descriptor file under the specified descriptor name. The application checks if the current (old) table schema is equal to the "new" schema and if true then do nothing, if false then patch.

This post will be updated with links to both source code (it's open source) and a dockerhub image.

## 3.3 Backup pipelines
In parallell with the generic pipelines we also have a backup pipeline that consumes messages from all topics and write the records to the same BigQuery table (partitioned on topic) as a data field (message as BYTES) together with message attributes and partitioned on ingestion time. Hence we can do a backfill at any point with a backfill pipeline that accepts a SQL-query and pubsub topic as parameters and point the output to the topic read by the generic pipeline for the specific topic, and will just scale up the workers during the backfill and go back to normal when done!

In the next post (part 4) I will focus on the automation part using Google Cloud Build.
