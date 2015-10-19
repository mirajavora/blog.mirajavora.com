---
layout: post
title: "Migrate SQL to Couchbase while keeping the lights on"
date: 2015-10-30 21:12:30 +0000
comments: true
image: /images/posts/statsd/metrics.png
summary: ""
categories: [devops, couchbase, mssql, datamigration]
---

When downtime is a no option, how do you safely migrate all your underlying data from one data store to another?
Especially when you are dealing with relational to NoSQL move?

I've done one almost exactly a year ago where failure was not an option.
<!--more-->


Migration Scenario
-------------------
A service in question was a RESTful API that provided significant data to various clients within an SOA architecture.
Through various dependencies it was serving over 1m requests per day - at peak times reaching 40 req/s with an avg response time of 30ms.

Availability and response time speed was crucial to the service, but it's underlying single SQL instance meant it was a single-source of failure.
We have decided to move to a Couchbase instance as the data was more suited to the document-store model.



Working out the migration plan
-------------------


### Write to both data-stores


### Copy over all docs from Couchbase to SQL


### Switch over reads to Couchbase, write to both


### Switch over writes to Couchbase only


Capacity Planning
-------------------
asdfasdf

