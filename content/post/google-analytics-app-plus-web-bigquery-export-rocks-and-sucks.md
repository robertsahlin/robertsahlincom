---
title: "Why Google Analytics App + Web BigQuery Export Rocks and Sucks"
date: 2019-09-02T13:45:30+01:00
draft: false
tags: ["BigQuery", "Google Analytics", "Firebase Analytics"]
---

Google recently released Google Analytics App + Web which essentially is something like Firebase Analytics for web (or Google Analytics version 2 if you want to). This is exciting for many reasons, two of them are:

* Google is finally moving away from a user-session-pageview based model to one built on users and events
* It supports BigQuery export also for standard users

That is awesome. These two are actually two of the primary reasons why I built datahem. Thanks Google, this rocks!

However, sadly I must point out, it also sucks and I'll tell you why. Notice that it is possible to get around many of the drawbacks listed below, but you will end up doing a lot of transformation in scheduling queries and maintain views and tables and you won't get it in streaming mode.

Google has chosen a data model that is commonly known as entity-attribute-value. This allows for very flexible schema.I.e. just log an event with (one or more) attribute/s and associated value/s and you will have it in BigQuery seconds later. A common issue with this pattern is that all values are logged as strings and it is up to the analyst or data engineer to know what data type it represents before being analysed/transformed. Google has tried to solve this partly by having a column for each data type, i.e. "value" : {"string_value" : "", "int_value" : , "float_value": , "double_value" : } but it still require the analyst/engineer to know where to look for a value. 

But this is not the only challenge. The schema is so flexible that the analyst/engineer doesn't even know what events (and associated attributes/values) are in the table without proper documenation or executing a query that returns event names and associated attributes/values, and to be sure you catch them all you need to scan all the partitions. Another challenge is that since all events have the exact same schema you can't set column based access rights on that table.

I can understand why Google chose this model, it is not uncommon for vendors that offer a generic solution to clients with very different entities. However, as a data engineer I would have preferred a solution where I can apply my own schema and define my own events and attributes.
