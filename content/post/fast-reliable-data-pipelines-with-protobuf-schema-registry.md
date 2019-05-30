---
title: "Fast and reliable data pipelines with protobuf schema registry"
date: 2019-05-31T7:50:33+01:00
draft: true
tags: ["DataHem", "Protobuf", "Apache Beam"]
---

My employer MatHem is growing quickly and so are the requirements for fast and reliable data pipelines. Since I joined a little more than one year ago I've been working hard on developing an event streaming platform (named DataHem) which meets those requirements, using Apache Beam and Protocol Buffers and running it on Google Cloud Platform.

MatHem is the biggest online grocery store in Sweden and to briefly explain how the business works:
1. A customer order groceries online in one of our digital channels (we have no physical stores).
2. The order is picked in one of our 3 warehouses located in the 3 biggest cities in Sweden, covering 65% of the Swedish population.
3. The groceries are loaded in one of our 200 cars (or 3 boats) and delivered to the customer (even to your jetty if you're in the Stockholm archipelago)

Data is collected in all of these steps as well as in the work of supply, pricing, product assortment, content production (recepies etc.).

The design goal has been to treat data as immutable streams (log) with strong contracts and publish data objects (orders, products, members, car temperatures, etc.) when they are created, modified or deleted. By doing so we meet business requirements on:
1. Scalability and complexity - Our quickly growing business also means our data platform needs to keep up with not only the ever-growing volumes of data but also the number of data sources and increasing complexity of our systems. 
2. No-ops - Also, the data engineering team is small (1,5 FTE) and we need to make sure that our time is efficiently used and adding new data sources requires a minimum of work to set up and maintain.
3. Synergies and down-stream dependencies - achieve synergies by collecting data once and use it for multiple purposes (reports, dashboards, analytics, machine learning and data products) and make sure that data products down-stream don't break when upstream data model evolves.
