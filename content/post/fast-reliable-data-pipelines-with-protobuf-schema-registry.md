---
title: "Fast and reliable data pipelines with protobuf schema registry"
date: 2019-05-31T7:50:33+01:00
draft: true
tags: ["DataHem", "Protobuf", "Apache Beam"]
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
2. No-ops and synergies - the data engineering team is small (1,5 FTE) and we need to make sure that our time is efficiently used and adding new data sources requires a minimum of work to set up and maintain. We also strive for synergies by collecting data once and use it for multiple purposes (reports, dashboards, analytics, machine learning and data products).
3. Data quality and resilience - make sure that data products down-stream don't break when the upstream data model evolves.

# Solution

## Data sources
Understanding the source of the data is fundamental. At MatHem, most business related data is produced by our IT development teams. Their data is extensively used by the Data Science team to discover business insights and to build data products that better serve our customers. Most of MatHem's frontend and backend is built on micro-services and runs on AWS and the developers use AWS SNS and Kinesis as their prefered message channels. These channels are some of the the most important data sources for us. We also have various clients (mobile apps & web) and third party services (webhooks) generating events data that are collected in realtime. However, the data that is passed around is in JSON format and 

To summarize, the following data sources need to be supported.

1. HTTP Requests (GET/POST)
2. AWS SNS
3. AWS Kinesis

## Data destinations
To satisfy our main use cases; reporting, adhoc/explorative analysis, visualizations and ML/Data products we land the processed data objects in our data warehouse (BigQuery) and as streams (PubSub). Authentication and authorization is set close to the data warehouse to allow users to use their tool of choice (if it supports BigQuery) for adhoc/explorative analysis. Hence, we need to set permissions on field level but also descriptions of each field. Think if we could apply a schema
