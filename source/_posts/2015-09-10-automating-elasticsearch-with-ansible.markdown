---
layout: post
title: "Automating Elastic Search deployments with Ansible"
date: 2015-03-20 21:12:30 +0000
comments: true
image: /images/posts/ansible/ansible-logo.png
summary: ""
categories: [ansible, elasticsearch, deployment, ubuntu, aws]
---

There is so much to love about Elastic Search. I've used Lucene several times in the past do to free-text search, but it was tough to built it with scalability and resilience in mind (possible but tougher than it needed to be).
Elastic Search abstracts all that work away, giving you a beautiful API with great tools on top of it.

If you haven't used it yet, check it out @elastic.
<!--more-->


Elastic Search
-------------------
Elastic Search is essentially a distributed lucene index over multiple nodes. Each index that you create within the elastic search consists of multiple shards that you define at index creation (default is 5).
Each shard can then have a specific number of replicas - these are copies of the data and should not live on the same node as the primary shard.

A shard is basically a single lucene index. When a request is made to an elastic search index, ES delegates this to each shard and aggregates the results.

### Plugins
There are also numerous plugins that make running and administering Elastic Search cluster super easy. There are so many so I wanted to call out only couple that I use always.

#### Kopf
I'm not quite sure what I would do without the [Kopf plugin](https://github.com/lmenezes/elasticsearch-kopf). It that handles everything from visibility of overall indexes, performance of the nodes and doc size to changing various cluster settings.
It can also run backups, snapshots, ad-hoc index queries or queries to change aliases. Did I mention it's awesome?

<img alt="Sendgrid Stats" src="/images/posts/sendgrid/elastic-search-kopf.png" class="" />


#### AWS Plugin
You cannot run Elastic Search on AWS without this plugin. It's as simple as that. It enabled unicast discovery and S3 storage (for backups and recovery).

[Check it out at the Github repo](https://github.com/elastic/elasticsearch-cloud-aws)

### Kibana & Other tools


Building the cluster using Cloudformation on AWS
-------------------
If you don't use AWS, feel free to skip this. However, if you do, this may decrease the time you spend creating the cluster - using Cloudformation.
Cloudformation essentially automates the creation of all your AWS resources starting from roles, EC2 instances, security groups and much more.

By using CF, you can be up and running within seconds and recreating the cluster repeatedly isn't a tedious job. Some may argue how hard
is it to build a two-three node cluster, but there's always scope for human error and it's not exactly time well spent - especially if it can be automated.

I created a fairly simple Cloudformation script that I've been re-using. It creates a multi-node auto-scaling group, specific role and a load-balander.

{% highlight bash %}


{% endhighlight %}

Deploying Elastic Search
-------------------
Once your infrastructure is setup, you're going to need to actually deploy elastic search and all the plugins to the nodes. That's where ansible can help you out massively.
If you've never used ansible, it's a neat way to run scripts on remote machines. [Check it out here](http://docs.ansible.com/ansible/intro_getting_started.html)

There is tun of open source scripts that help you run different ansible roles. Creating elastic search cluster is one of them.
The one I've been using is [TODO]

### Changing variables


### Running the script

