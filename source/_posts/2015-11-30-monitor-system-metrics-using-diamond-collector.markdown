---
layout: post
title: "Monitor System Metrics using Diamond Collector"
date: 2015-11-30 10:12:30 pm
comments: true
image: /images/posts/sendgrid/sendgrid-stats-small.png
summary: "If you're looking for a tool to collect and push system level metrics into graphite, Diamond should be your first port of call. "
categories: [diamond, metrics, graphite, grafana, python, statsd]
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

All these metrics should be part of your business metrics that you verify when doing releases. A common approach is to have a canary node(s) that you deploy the latest version. Once deployed, you can verify these metrics compared to the rest of the cluster before deploying on all nodes.
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
In most cases you are unlikely to use them all, but here are some that I would like to highlight

- [Memory collector](https://github.com/python-diamond/Diamond/wiki/collectors-MemoryCollector)
- [CPU collector](https://github.com/python-diamond/Diamond/wiki/collectors-CPUCollector)
- [Available disk space collector](https://github.com/python-diamond/Diamond/wiki/collectors-DiskSpaceCollector)


### Setting up diamond
I prefer to have them running as part of my [ansible scripts](http://docs.ansible.com/ansible/intro_getting_started.html).


#### Install Diamond via ansible
Depening on how your code is structured, I tend to add a role that takes care of installing all the dependencies, adding the configuration and any custom collectors I may have.

{% gist b85b0606591bbc8ad90e diamond.yml %}

Your dir structure of the role should also look something along these lines

{% highlight bash %}

- tasks
  - main.yml
- files
  - collectors
    - SomeCustomCollector1.py
    - SomeCustomCollector2.py
- templates
   - diamond.conf

{% endhighlight %}

#### Configuration
The role relies on diamond configuration file. The entire file can be found [here](https://gist.github.com/mirajavora/9f6b5cd402c5c6f27df5), but I can pick out few key properties to watch out for.

{% highlight bash %}

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

{% endhighlight %}


Building custom collectors
-------------------

