---
layout: post
title: "Monitor System Metrics using Diamond Collector"
date: 2016-02-02 10:12:30 pm
comments: true
thumbnail: /images/posts/diamond/diamond.png
summary: "If you're looking for a tool to collect and push system level metrics into graphite, Diamond should be your first port of call.  It includes a huge range of various collectors that lets you monitor anything from cpu, memory and disk space to performance of elastic search or haproxy."
tags: [diamond, metrics, graphite, grafana, python, statsd]
archive: 2016
---

I while back I blogged about how to approach [error logging in production multi-node environments](/scaling-logging-logstash-and-graylog2/) so that you stay sane.
Later on, I looked at a way to [deal with application level metrics using statsd](/logging-custom-metrics-with-statsd/).

One last piece of the puzzle is monitoring of the health of the nodes themselves. This typically includes CPU and Memory usage, disk space monitoring and application health monitoring.
Diamond collectors are great exactly for that.
<!--more-->


Monitoring your server health
-------------------
I've seen instances where a server has gone down purely because it ran out of disk-space. It's also not uncommon to have a rogue process that suddenly takes up all the CPU in spikes that are easy to miss if you don't have the right visibility.
It's also a good idea to measure the available memory on your servers - in case you hit a slow memory leak that manifests itself over-time.

All these metrics should be part of your business metrics that you verify when doing releases. A common approach is [to have a canary node(s) that you deploy the latest version](http://martinfowler.com/bliki/CanaryRelease.html). Once deployed, you can verify these metrics compared to the rest of the cluster before deploying on all nodes.
In an ideal world, these checks should be automated and be part of promoting your builds.


Diamond collectors
-------------------
There's a variety of ways to get these metrics from unix boxes to something like graphite.
My choice has always been Diamond - mainly for it's simplicity to set up.

### Diamond
Diamond is a python daemon that collects system metrics and publishes them to Graphite or other supported handlers.
It is capable of collecting cpu, memory, network, i/o, load and disk metrics.
Additionally, it features an API for implementing custom collectors for gathering metrics from almost any source.

It was originally made by BrightCove, but now it's maintained mainly by devs from python-diamond.

You [can find it on Github](https://github.com/python-diamond/Diamond).


### Collectors
Diamond contains a [ton of different collectors](https://github.com/python-diamond/Diamond/wiki/Collectors).
While you have a ton to choose from, you are unlikely to need them all. The active collectors are defined via config. Here are some that I would like to highlight

- [Memory collector](https://github.com/python-diamond/Diamond/wiki/collectors-MemoryCollector)
- [CPU collector](https://github.com/python-diamond/Diamond/wiki/collectors-CPUCollector)
- [Available disk space collector](https://github.com/python-diamond/Diamond/wiki/collectors-DiskSpaceCollector)



### Setting up diamond
There are multiple ways to get diamond set-up. The repo illustrates [few manual steps to get it running on a box](https://github.com/python-diamond/Diamond#getting-started).
However, I prefer to have them running as part of my [ansible scripts](http://docs.ansible.com/ansible/intro_getting_started.html) so that it can be repeated and automated effortlessly.


#### Install Diamond via ansible
Depening on how your code is structured, I tend to add a diamond role that takes care of installing all the dependencies, adding the configuration and any custom collectors I may have.

{% gist b85b0606591bbc8ad90e diamond.yml %}

Your dir structure of the role should also look something along these lines

{{< highlight bash "linenos=inline" >}}

- tasks
  - main.yml
- files
  - collectors
    - SomeCustomCollector1.py
    - SomeCustomCollector2.py
- templates
   - diamond.conf

{{< / highlight >}}

#### Configuration
The role relies on a diamond configuration file. This effectively specifies what handler we're going to use (in this instance graphite), what collectors are enabled and their settings and the intervals in which they are collected.
The entire file can be found [here](https://gist.github.com/mirajavora/9f6b5cd402c5c6f27df5), but I can pick out few key properties to watch out for.

{{< highlight bash "linenos=inline" >}}

# Specify that metrics are sent via graphite handler
handlers = diamond.handler.graphite.GraphiteHandler

....

# Interval when the metrics are collected and sent off
collectors_reload_interval = 3600

....

# Graphite server host
host = {{ graphite_host }}

# Port to send metrics to
port = 2003

...
# Specify path prefix or suffix
path_prefix = your-metrics
path_suffix = suff

{{< / highlight >}}


Building custom collectors
-------------------
While Diamond comes with a long list of in-built collectors, it also allows you to build your custom collectors. We've done this recently as we needed to monitor an endpoint on localhost and push that as a metric intro graphite.
Rather than building a cron job, custom collector was perfect for that use-case. I conviniently lifted this from the original repo.

Your collector needs to inherit from the **diamond.collector.Collector** and implement the **collect** method. You can [check out the base class here](https://github.com/python-diamond/Diamond/blob/master/src/diamond/collector.py) for more info on what's exposed to you while writing the collector.

{% gist af8a2298542b8a23d694 HealthCheckCollector.py %}

There is also an [example collector on the github page of the project](https://github.com/python-diamond/Diamond/blob/master/src/collectors/example/example.py).