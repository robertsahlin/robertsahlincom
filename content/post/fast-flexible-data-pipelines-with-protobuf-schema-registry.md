---
title: "Fast and flexible data pipelines with protobuf schema registry"
date: 2019-05-31T7:50:33+01:00
draft: true
tags: ["DataHem", "Protobuf", "Apache Beam", "BigQuery", "Dataflow"]
---

MatHem is growing quickly and so are the requirements for fast and reliable data pipelines. Since I joined the company a little more than one year ago I've been developing an event streaming platform (named DataHem) to meet those requirements. 

# Context
MatHem is the biggest online grocery store in Sweden and to briefly give a context this is how the business works:
1. A customer orders groceries online in one of our digital channels (we have no physical stores).
2. The order is picked in one of our 3 warehouses located in the 3 biggest cities in Sweden, covering 65% of the Swedish population.
3. The groceries are loaded in one of our 200 cars (or 3 boats) and delivered to the customer (even to your jetty if you're in the Stockholm archipelago)

Data is generated and collected in all of these steps as well as in the areas of supply, pricing, product assortment, content production, etc. Hence, data is embraced as a first-class citizen at MatHem and critical to generate advantages in a competitive market.

# Requirements
A major design goal has been to treat data as streams of immutable data objects (log) and publish the objects (orders, products, members, car temperatures, etc.) when they are created, modified or deleted. The data pipelines apply strong contracts early in the process. Following this pattern enable us to meet business requirements on:
1. Scalability and flexibility - Our quickly growing business also means our data platform needs to keep up with not only the ever-growing volumes of data but also the number of data sources and increasing complexity of our systems. 
2. No-ops and synergies - the data engineering team is small and we need to make sure that our time is efficiently used and adding new data sources requires a minimum of work to set up and maintain. We also strive for synergies by collecting data once and use it for multiple purposes (reports, dashboards, analytics, machine learning and data products).
3. Data quality and resilience - make sure that data products down-stream don't break when the upstream data model evolves.

# Solution
In order to meet the requirements we've built DataHem, a ... At the center of DataHem solution architecture are protobuf schemas and a generic dataflow pipeline. 

## Data sources
Understanding the source of the data is fundamental. At MatHem, most business related data is produced by our IT development teams. Their data is extensively used by the Data Science team to discover business insights and to build data products that better serve our customers. Most of MatHem's frontend and backend is built on micro-services and runs on AWS and the developers use AWS SNS and Kinesis as their prefered message channels. These channels are some of the the most important data sources for us. We also have various clients (mobile apps & web) and third party services (webhooks) generating events data that are collected in realtime. However, the incoming data is in JSON format and we want strong contracts early on. Hence, we transform the JSON to protobuf that is available in many programming languages, has efficient serialization and supports schema evolution. All data objects also get meta data such as source, UUID and a timestamp as attributes.

To summarize, the following data sources need to be supported:

1. HTTP Requests (GET/POST)
2. AWS SNS
3. AWS Kinesis

## Data collection
We use different GCP services as data collectors, all of them publish the JSON payload and meta data as pubsub messages. Each message type has its own pubsub topic and each topic has two subscriptions, one for backup and one for enrichment. 

#### AWS Kinesis collector
For high volume streams from AWS we use dataflow/Apache Beam to read from AWS Kinesis. AWS user and secret are encrypted in GCP KMS to avoid accidental exposure of credentials.

#### AWS SNS HTTP Subscriber collector
For lower volume and small bursts of data from AWS we use cloud functions to read data from AWS SNS. The cloud function collector routes the incoming messages based on a query parameter, hence we can reuse the same cloud function endpoint for many HTTP SNS subscriptions and minimize operations.

#### HTTP collector
For data that is collected as HTTP GET or POST requests we use AppEngine standard. Our HTTP collector enable us to collect data from user interactions on our web application (we send all Google Analytics hits to our HTTP collector) and from third party service webhooks (like survey monkey forms). 

## Data processing
We use dataflow to process streaming data from pubsub and write to BigQuery in streaming mode. Many of the data sources are processed in the same way, it’s just the schema that differs. Hence we’ve built a generic dataflow pipeline and specify processing logic in protobuf schemas using message and field options. One of our field level option is BigQuery field categories which allow us to set access control already in the schemas. Dataflow picks up the schema from a schema registry hosted in cloud storage (and built with cloud build triggered by schema updates in GitHub) and create the BigQuery table (if not already exists) before writing data to it. Hence, adding a new data source and streaming data to a BigQuery table with right field level access control is done by adding a protobuf schema to GitHub.

## Data destinations
To satisfy our main use cases; reporting, adhoc/explorative analysis, visualizations and ML/Data products we write the processed data objects to our data warehouse (BigQuery) and as streams (PubSub). Authentication and authorization of users is done close to the data warehouse to allow users to use their tool of choice (if it supports BigQuery) for adhoc/explorative analysis. Different roles in the company have different permissions to data Hence, we need to set permissions on field level but also descriptions of each field. Think if we could apply a schema



## Data backup
All raw data is streamed to a BigQuery dataset using dataflow jobs. The backup data is partitioned by ingestion time and have meta data (source, UUID, timestamp, etc.) in an attributes map that makes it easy to locate unique rows without parsing the raw data field (BYTES[]). Then a backfill is just a dataflow batch job that accepts a SQL-query and publish the data on the pubsub to be consumed by the streaming processing job together with the current data. The backfill data includes a message attribute that informs that it is a backfill entity and hence is filtered out from being written to the backup table again.

# Summary
If you're a data engineer and find this post interesting, don't hesitate to reach out knowing that we've open data engineer positions at MatHem.
