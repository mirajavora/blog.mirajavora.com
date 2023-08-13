---
layout: post
title: "Logging application metrics with StatsD"
date: 2015-03-10 19:12:30 +0000
comments: true
thumbnail: /images/posts/statsd/metrics.png
summary: "If you don't track it, you can't measure it. Realtime service and business metrics should be part of any production application.  Knowing how is the app performing is as important as measuring whether the product impact of your changes."
tags: [statsd, metrics, graphite, etsy]
archive: 2015
aliases:
    - /logging-custom-metrics-with-statsd/
---

Application monitoring and service metrics
-------------------

*If you don't track it, you can't measure it*. Realtime service and business metrics should be part of any production application.
Knowing how is the app performing is as important as measuring whether the product impact of your changes.

A good set of service metrics lets you effectively monitor the impact of your changes on the app performance. 
Has your change to multi-threading really achieved the times x throughput? Has the last dependency injection change caused a slow memory leak?
Are there levels of response codes you're not expecting in your app?
<!--more-->

There are a variety of ways to deal with application level metrics. The world of .NET offers you perf counters while unix has a variety of possibilities. 
Usuaully, I would go for the **simplest, most friction-less option**. That's why StatsD with Graphite is so appealing.


StatsD
-------------------

Etsy talks about **[measuring everything and anaything](https://codeascraft.com/2011/02/15/measure-anything-measure-everything/)**.
Their real obsession with metrics made them release an awesome library that took great traction - because it's simple and effective.

In essence, [StatsD](https://github.com/etsy/statsd/) server listens on a UDP/TCP port and collects metrics. 
These are then aggregated and in intervals passed to a back-end of your choice - most likely graphite.
 
The advantage of the community is that there is a massive list of client implementations so integrating StatsD into your app is super-straightforward.
There are multiple clients for node, python, java, ruby, php, .net, go .... and more. [Check out the entire list](https://github.com/etsy/statsd/wiki).
Also, you will find a bunch of server implementations beyond the original Node.js. If you fancy StatsD on windows machines, check out [statsD.net](https://github.com/lukevenediger/statsd.net).


Integrate StatsD client into your JVM app
-------------------

If you're looking to add statsD into your JVM application, the [java client by tim group](https://github.com/tim-group/java-statsd-client) seemed like the best choice.
It has zero dependencies and it's pretty straight-forward.

Add your dependency to mvn or gradle

{{< highlight xml "linenos=inline" >}}

MVN

    <dependency>
        <groupId>com.timgroup</groupId>
        <artifactId>java-statsd-client</artifactId>
        <version>3.0.1</version>
    </dependency>

Gradle
    'com.timgroup:java-statsd-client:3.1.0'

{{< / highlight >}}

And init the statsd client with the prefix, host of the statD server and the port. 
StatsD has a concept of namespaces, where you can group your metrics - that allows for better visualisation and keeps them neat. The choice of the namespace is yours, depending on what suits you. 
In bigger deployments you might go for something like "application-name.data-centre.box-name.counter-name". 

{{< highlight java "linenos=inline" >}}

    import com.timgroup.statsd.StatsDClient;
    import com.timgroup.statsd.NonBlockingStatsDClient;
    
    public class DiagnosticsService {
        private static final StatsDClient statsd;
    
        public DiagnosticsService(String host, int portNumber) {
            statsd = = new NonBlockingStatsDClient("your.custom.prefix", host, portNumber);
        }
        
        .....
    }
    
{{< / highlight >}}

I tend to have a single statsD client within the app as a singleton wrapped by a diagnostics service.

StatsD Metric Types
-------------------

StatsD supports a range of metric types. These you fit 99% of your metric logging scenarios.
It also has a concept of a flush interval, where the data is sent off to back-ends.

### Counters
Basic counters that are incremented each time you log against the counter. These are reset to 0 at flush.
You can also set a sampling interval to tell StatsD you're only sending part of the data-set.

{{< highlight bash "linenos=inline" >}}
    your.namespace.counter:1|c
{{< / highlight >}}


### Timers
These are great for monitoring response times of any kind. You tell statsD how long an action took.
It then automatically works out percentiles, average (mean), standard deviation, sum, and min/max. Really awesome.

{{< highlight bash "linenos=inline" >}}
    your.namespace.response_time:300|ms
{{< / highlight >}}

### Gauges
Gauges are single values that can be incremented or decremented or set to a specific value. Unlike counters, gauges aren't reset to zero at flush time


### Sets
These count unique set of occurrences between flushes.



Logging metrics using the JVM client
-------------------

Using the  [JAVA implementation of the StatsD client](https://github.com/tim-group/java-statsd-client) is then pretty straight-forward.


{{< highlight java "linenos=inline" >}}

    import com.timgroup.statsd.NonBlockingStatsDClient;
    import com.timgroup.statsd.StatsDClient;


    public final class StatsDPerformanceService implements DiagnosticsService {

        private static StatsDClient statsd = null;
        private static DiagnosticsConfig config;

        public StatsDPerformanceService(DiagnosticsConfig configuration) {
            config = configuration;
            statsd = new NonBlockingStatsDClient(
                    getPrefix(), config.getHost(), config.getPort());
        }

        private String getPrefix() {
            return String.format("yourprefix.%s", config.getBoxName()).toLowerCase();
        }

        @Override
        public void incrementCounter(String counterName) {
            if(config.getEnableMetrics())
                statsd.incrementCounter(counterName);
        }

        @Override
        public void decrementCounter(String counterName) {
            if(config.getEnableMetrics())
                statsd.decrementCounter(counterName);
        }

        @Override
        public void gauge(String gaugeName, long value) {
            if(config.getEnableMetrics())
                statsd.gauge(gaugeName, value);
        }

        @Override
        public void recordExecutionTime(String timerName, long value) {
            if(config.getEnableMetrics())
                statsd.recordExecutionTime(timerName, value);
        }
    }

{{< / highlight >}}

you can then also consider helper methods using runnable and callable to wrap timings around the methods

{{< highlight java "linenos=inline" >}}


        @Override
        public <T> T executeWithTimer(Callable<T> callable, String counterName) {
            if(callable == null || counterName == null)
                return null;

            T result = null;
            long startTime = System.nanoTime();
            try {
                result = callable.call();
            } catch (Exception e) {
                e.printStackTrace();
            }
            long endTime = System.nanoTime();

            long duration = (endTime - startTime)/1000000;

            recordExecutionTime(counterName, duration);
            return result;
        }

    ... and execute like

    String result = diagnosticsService.executeWithTimer(
    () -> randomService.getResult(someVar), "SuperAwesomeNameOfTheCounter");

{{< / highlight >}}

Enjoy! StatsD is great - I'll look at configuring StatD and graphite in my next post.