---
layout: post
title: "Logging metrics with StatsD on JVM"
date: 2015-03-10 19:12:30 +0000
comments: true
image: /images/posts/logstash/logstash.png
summary: ""
categories: [StatsD, metrics, graphite, Logstash, Elasticsearch, Log4j]
---

Application monitoring and service metrics
-------------------

If you don't track it, you can't measure it. Realtime service and business metrics should be part of any production application. 
Knowing how is the app performing is as important as measuring whether the product impact of your changes.

A good set of service metrics lets you effectively monitor the impact of your changes on the app performance. 
Has your change to multi-threading really achieved the times x throughput? Has the last dependency injection change cause a slow memory leak? 
Why is there of 404s in your app?

There are a variety of ways to deal with application level metrics. The world of .NET offers you perf counters while unix has a variety of possibilities. 
Usuaully, I would go for the simplest, most friction-less option. That's why StatsD with Graphite is so appealing.


StatsD
-------------------

Etsy talks about [measuring everything and anaything](https://codeascraft.com/2011/02/15/measure-anything-measure-everything/). 
Their real obsession with metrics made them release an awesome library that took great traction - because it's simple and effective.

In essence, [StatsD](https://github.com/etsy/statsd/) server listens on a UDP/TCP port and collects metrics. 
These are then aggregated and in intervals passed to a back-end of your choice - most likely graphite.
 
The advantage of community is that there is a massive list of client implementations so integrating StatsD into your app is super-straightforward.
There are multiple clients for node, python, java, ruby, php, .net, go .... and more. [Check out the entire list](https://github.com/etsy/statsd/wiki).
Also, you will find a bunch of server implementations beyond the original Node.js. If you fancy StatsD on windows machines, check out [statsD.net](https://github.com/lukevenediger/statsd.net).


Integrate StatsD
-------------------

Each Logstash instance can be configured with multiple inputs and outputs and the topology very much depends on your use cases. 
For example, you may chose to run a single centralised instance of logstash with configs pointing to each separate box or on the contrary, run logstash on every instance.

![Different approaches to deploying Logstash](/images/posts/logstash/logstash-diagram.png)

Did I meantion Logstash has the best logo in the universe?


Log your metrics
-------------------

In my scenario, I was looking to collect multiple logs from multiple apps on a large number of boxes across multiple data-centres.
Each of the apps already had logging, which was mostly based on log4net. 

Rather than having a separate configuration for each logstash instance, it was much simpler to use the *UDP plugin* from logstash.
This means the existing apps just added a new appender that would log to an UDP port and logstash would just pick that up.

This made the whole logstash config really simple

{% highlight json %}
    input { 
        udp { 
          port => 5960 
          codec => plain { 
            charset => "UTF-8" 
          } 
          type => "log4net" 
        }
    }
{% endhighlight %}

Changes to the Log4net appender were straight-forward too.

{% highlight xml %}
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

{% endhighlight %}


Adding Graylog2 into the mix
-------------------

[Graylog2](https://www.graylog2.org/) is one of my favourite tools. The sole purpose is to aggregate and analyse logs in real time. With elastic search sitting underneath,
it let's you do complex queries on the data and create custom dashboards. 

<a href='/images/posts/logstash/screen2_full.png'><img src='/images/posts/logstash/screen2.png' alt='Graylog2' /></a>
<a href='/images/posts/logstash/screen3_full.png'><img src='/images/posts/logstash/screen3.png' alt='Graylog2' /></a>

If you have an instance of Graylog2 running somewhere, it's easy to use the gelf output to channel all the incoming logs into Logstash into Graylog2.
The full logstash config could then look something like this:

{% highlight json %}

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

{% endhighlight %}

A result will be a centralised stream of logs that you can easily analyse and create dashboards from. 
The beauty of logstash is that you can easily add further outputs (eg graphite). 
