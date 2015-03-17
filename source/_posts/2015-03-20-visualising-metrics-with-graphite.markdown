---
layout: post
title: "Visualising Metrics With Graphite"
date: 2015-03-20 21:12:30 +0000
comments: true
image: /images/posts/statsd/metrics.png
summary: ""
categories: [graphite, metrics, grafana, etsy]
---

Graphite is by far the best tool that I've used to record and visualise metrics.
I've tried several flavours ofsystem  metrics visualisation tools over the course my programming years, but none felt so straightforward, easy to use and yet so powerful.

There's no wonder that big players like etsy [TODO] use graphite for the mission critical graphing.
<!--more-->


Graphite under the hood
-------------------
Graphite was built in python and consists of several components make up the full package. They are carbon, whisper and web.

![Graphite Architecture](/images/posts/graphite/overview.png)

### Carbon
Is a set of daemons that run and effectively listen to the metrics coming in.
In a simple setup, you are most likely going to need only *carbon-cache.py*. You can utilise the other daemons in more complex scenarios where you need data replication and higher I/O performance.

### Whisper
Carbon hands off the data to whisper back-end, which is a fixed-size file based database.
*Whisper allows for higher resolution (seconds per point) of recent data to degrade into lower resolutions for long-term retention of historical data.*

### Graphite Web
Finally, there is a web component that renders the graphs based on the metrics.


Installing Graphite
-------------------
Yes, it's not as straight-forward as pulling down a single package, but it's worth it!
First, there's an obvious on Python - graphite supports 2.6 and higher.





Getting your metrics into the graphing server
-------------------

### Aggregation

### Functions



Visualising metrics and getting most out of grafana
-------------------
Graphite offers an in-built web project to visualise the metrics. It's good and lot of people like it. You can build up your graphs using


Meet Grafana
-------------------
asdf