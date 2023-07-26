
---
layout: post
title: "Running Logstash as a Service"
date: 2014-12-26 08:32:30 +0000
comments: true
image: /images/posts/logstash/logstash.png
summary: ""
categories: [Logging, Log4Net, Logstash, Elasticsearch]
---

In a previous post, I looked at utilising Logstash to collect various types of logs and centralise the analysis in something like graylog2 for visibility.

As Logstash is JVM based (actually in JRuby), it can run pretty much anywhere JVM, which is awesome.
This posts looks at setting up Logstash on a Windows machine as a service.
<!--more-->

Java Service Wrapper
-------------------

You can't the Logstash JAR directly - the best alternative is to use a JAVA Service wrapper that will look after the windows service lifecycle for you.

As you scale and increase the amount of boxes and systems that you manage, this all becomes a lot more challenging and it's easy to lose visibility. 
If you manage 30 boxes, you're not going to check every single one of them for logs - it's cumbersome and too painful.
The chance of missing a defect or a misbehaving service also grows.

Logstash
-------------------

This is where [Logstash](http://logstash.net/) comes in, it's an open-source framework to manage events and logs.
In a nutshell, Logstash can take a variety of inputs, apply filters and transformations on the data that comes in and then push them forward to another store.

As an open source project, it contains a massive amount of plugins. Just to name a few

 - Inputs - collectd, raw files, elasticsearch, gelf, unix, s3, redis, rabbitmq
 - Outputs - cloudwatch, boundary, elasticsearch, gelf, graphite, sdout
 - Codecs - JSON, dots, multiline, fluent, graphite and many more

To check out a full list, [go to the logstash documentation](http://logstash.net/docs/1.4.2/).


Configuring Logstash
-------------------

Each Logstash instance can be configured with multiple inputs and outputs and the topology very much depends on your use cases. 
For example, you may chose to run a single centralised instance of logstash with configs pointing to each separate box or on the contrary, run logstash on every instance.

![Different approaches to deploying Logstash](/images/posts/logstash/logstash-diagram.png)

Did I meantion Logstash has the best logo in the universe?


Log4net/Log4j and Logstash
-------------------

In my scenario, I was looking to collect multiple logs from multiple apps on a large number of boxes across multiple data-centres.
Each of the apps already had logging, which was mostly based on log4net. 

Rather than having a separate configuration for each logstash instance, it was much simpler to use the *UDP plugin* from logstash.
This means the existing apps just added a new appender that would log to an UDP port and logstash would just pick that up.

This made the whole logstash config really simple

{{< highlight bash "linenos=inline" >}}
TODO
{{< / highlight >}}

Changes to the Log4net appender

{{< highlight bash "linenos=inline" >}}
TODO
{{< / highlight >}}


Adding Graylog2 into the mix
-------------------

[Graylog2](https://www.graylog2.org/) is one of my favourite tools. The sole purpose is to aggregate and analyse logs in real time. With elastic search sitting underneath,
it let's you do complex queries on the data and create custom dashboards. 

<a href='/images/posts/logstash/screen2_full.png'><img src='/images/posts/logstash/screen2.png' alt='Graylog2' /></a>
<a href='/images/posts/logstash/screen3_full.png'><img src='/images/posts/logstash/screen3.png' alt='Graylog2' /></a>

If you have an instance of Graylog2 running somewhere, it's easy to use the gelf output to channel all the incoming logs into Logstash into Graylog2.

{{< highlight bash "linenos=inline" >}}
TODO full logstash config

{{< / highlight >}}

A result will be a centralised stream of logs that you can easily analyse and create dashboards from. 
The beauty of logstash is that you can easily add further outputs (eg graphite). 

Related Articles
-------------------

[Logstash and GrayLog2 - Scaling your Logging](/introduction-to-sass-with-visual-studio/)<br/>