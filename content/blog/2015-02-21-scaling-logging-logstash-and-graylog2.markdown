---
layout: post
title: "Logstash and GrayLog2 - Scaling your Logging"
date: 2015-02-21 21:52:30 +0000
comments: true
thumbnail: /images/posts/logstash/logstash.png
summary: "As you scale and increase the amount of boxes and systems that you manage visibility and centralised logging becomes crucial. Logstash and Graylog2 are the perfect combo to tackle this problem. If you're interested in logging at scale, read on ;-)"
tags: [Logging, Log4Net, Graylog2, Logstash, Elasticsearch, Log4j]
archive: 2015
aliases:
  - /scaling-logging-logstash-and-graylog2
---

Logging visibility when scaling
-------------------

Every decent app produces some kind of logging. Traditionally, this has been achieved by wrapper such as [log4net](http://logging.apache.org/log4net/), [slf4j](http://www.slf4j.org/) [log4j](http://logging.apache.org/log4j/2.x/) and many others.
The information it produces is invaluable as it's usually the only source of information when troubleshooting issues.
<!--more-->

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

{{< highlight json "linenos=inline" >}}
    input { 
        udp { 
          port => 5960 
          codec => plain { 
            charset => "UTF-8" 
          } 
          type => "log4net" 
        }
    }
{{< / highlight >}}

Changes to the Log4net appender were straight-forward too.

{{< highlight xml "linenos=inline" >}}
... 
        <appender name="UdpAppender" type="log4net.Appender.UdpAppender">
            <RemoteAddress value="127.0.0.1" /> <!-- set to 127.0.0.1 and host name mapped to this on my machine (port 80) -->
            <RemotePort value="5960" />
            <layout type="log4net.Layout.PatternLayout">
                <conversionPattern value="%date [%thread] %-5level - %property{log4net:HostName} - MyApplication - %message%newline" />
            </layout>
        </appender>
....
add the appender to the root loggers
....
		<root>
			<level value="ERROR" />
			<appender-ref ref="OutputDebugStringAppender" />
			<appender-ref ref="TraceAppender" />
			<appender-ref ref="ErrorFileAppender" />
			<appender-ref ref="UdpAppender" />
		</root>

{{< / highlight >}}


Adding Graylog2 into the mix
-------------------

[Graylog2](https://www.graylog2.org/) is one of my favourite tools. The sole purpose is to aggregate and analyse logs in real time. With elastic search sitting underneath,
it let's you do complex queries on the data and create custom dashboards. 

<a href='/images/posts/logstash/screen2_full.png'><img src='/images/posts/logstash/screen2.png' alt='Graylog2' /></a>
<a href='/images/posts/logstash/screen3_full.png'><img src='/images/posts/logstash/screen3.png' alt='Graylog2' /></a>

If you have an instance of Graylog2 running somewhere, it's easy to use the gelf output to channel all the incoming logs into Logstash into Graylog2.
The full logstash config could then look something like this:

{{< highlight json "linenos=inline" >}}

input { 
	udp { 
	  port => 5960 
	  codec => plain { 
		charset => "UTF-8" 
	  } 
	  type => "log4net" 
	}
}


filter {
  if [type] == "log4net" {
    grok {
      remove_field => message
      match => { message => "%{TIMESTAMP_ISO8601:sourceTimestamp} \[%{NUMBER:threadid}\] %{LOGLEVEL:loglevel} +- %{IPORHOST:tempHost} - %{GREEDYDATA:tempMessage}" }
    }
    if !("_grokparsefailure" in [tags]) {
      mutate {
        replace => [ "message" , "%{tempMessage}" ]
        replace => [ "host" , "%{tempHost}" ]
      }
    }
    mutate {
      remove_field => [ "tempMessage" ]
      remove_field => [ "tempHost" ]
    }
  }
}

output {
  gelf {
	 host => "your-host-name"
	 custom_fields => ["environment", "PROD", "service", "BestServiceInTheWorld"]
	 }
  stdout { }
}

{{< / highlight >}}

A result will be a centralised stream of logs that you can easily analyse and create dashboards from. 
The beauty of logstash is that you can easily add further outputs (eg graphite). 
