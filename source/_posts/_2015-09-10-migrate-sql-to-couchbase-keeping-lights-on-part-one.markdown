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


Installing Graphite and NGINX on Ubuntu
-------------------
Yes, it's not as straight-forward as pulling down a single package, but it's worth it!
First, there's an obvious on Python - graphite supports 2.6 and higher.

### Pulling down dependencies

Luckily, there's an apt-get and most of the dependencies are in repos.

{% highlight bash %}

    apt-get update
    apt-get install graphite-web graphite-carbon

{% endhighlight %}

These pull down the key dependencies. In order to run the graphite-web, you'll need a web-server as well to serve the requests.
The two obvious choices are Apache or Nginx - I went for the latter.

{% highlight bash %}

    apt-get install nginx
    apt-get install build-essential python-dev

{% endhighlight %}


### Configuring carbon
The next step is to configure graphite daemons and nginx. First, let's configure carbon - the collector daemon.

{% highlight bash %}

    sudo nano /etc/default/graphite-carbon

{% endhighlight %}

Change CARBON_CACHE_ENABLED from false to true if you want the daemon to start at startup ..

{% highlight bash %}

    CARBON_CACHE_ENABLED=true

{% endhighlight %}

### Configuring nginx and uwsgi

First create a graphite site configuration in the nginx sites-available dir and link it to the sites enabled folder.

{% highlight bash %}

    nano /etc/nginx/sites-available/graphite

    ... create a link to sites enabled

    ln -s /etc/nginx/sites-available/graphite /etc/nginx/sites-enabled/

{% endhighlight %}

Open the graphite file and add the configuration.

{% highlight json %}

    server {
      listen 8080;
      charset utf-8;
      access_log /var/log/nginx/graphite.access.log;
      error_log /var/log/nginx/graphite.error.log;

      location / {
      include uwsgi_params;
      uwsgi_pass 127.0.0.1:3031;
      }
    }

{% endhighlight %}

Finally, setup uwsgi to enable the python app running under nginx.
If you want to read a bit more about uwsgi, there's a good [quickstart here](http://uwsgi-docs.readthedocs.org/en/latest/WSGIquickstart.html).


[uwsgi]
processes = 2
socket = 127.0.0.1:3031
gid = www-data
uid = www-data
chdir = /opt/graphite/conf
module = wsgi:application


apt-get install uwsgi-core


sudo graphite-manage syncdb


[TODO] - mention the DB

Getting your metrics into the graphing server
-------------------

Once you get your graphite server up and running, you can start pushing metrics into it. You can do that directly,
however, it's more likely you will utilise one of the many metric collection services available.

If you're looking for inspiration, check out StatsD for collecting service metrics,
Diamond for machine metrics and logstash for processing logging metrics.


Visualising metrics using graphite web project
-------------------
Graphite offers an in-built web project to visualise the metrics. It's decent and lot of people like it.
You can build up your graphs using

## Aggregation


## Functions


Meet Grafana - a better way to visualise metrics
-------------------
asdf