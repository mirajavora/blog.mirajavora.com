---
layout: post
title: "Automating Elastic Search deployments with Ansible on AWS"
date: 2015-09-18 09:12:30 +0000
comments: true
thumbnail: /images/posts/ansible/ansible-logo.png
summary: "There is so much to love about Elastic Search. If you, like me, end up creating more than one cluster, you start to think about automating the task whole task including infrastructure. Cloudformation and ansible scripts can make this tedious job super simple."
tags: [ansible, elasticsearch, deployment, ubuntu, aws]
archive: 2015
aliases:
  - /automating-elasticsearch-with-ansible-on-aws/
---

There is so much to love about Elastic Search. If you, like me, end up creating more than one cluster, you start to think about automating the task whole task including infrastructure.

Cloudformation and ansible scripts can make this tedious job super simple.
<!--more-->


Elastic Search
-------------------
I've used Lucene several times in the past do to free-text search, but it was tough to built it with scalability and resilience in mind (possible, but tougher than it needed to be).
Elastic Search abstracts all that work away, giving you a beautiful API with great tools on top of it.

If you haven't used it yet, check it out [@elastic](https://www.elastic.co/products/elasticsearch).

It is essentially a distributed lucene index over multiple nodes. Each index that you create within the elastic search consists of multiple shards that you define at index creation (default is 5).
Each shard can then have a specific number of replicas - these are copies of the data and should not live on the same node as the primary shard.

A shard is basically a single lucene index. When a request is made to an elastic search index, ES delegates this to each shard and aggregates the results.

### Plugins
There are also numerous plugins that make running and administering Elastic Search cluster easy. There are so many so I wanted to call out only couple that I use always.

#### Kopf
I'm not quite sure what I would do without the [Kopf plugin](https://github.com/lmenezes/elasticsearch-kopf). It that handles everything from visibility of overall indexes, performance of the nodes and doc size to changing various cluster settings.
It can also run backups, snapshots, ad-hoc index queries or queries to change aliases. Did I mention it's awesome?

<img alt="Sendgrid Stats" src="/images/posts/sendgrid/elastic-search-kopf.png" class="" />


#### AWS Plugin
You cannot run Elastic Search on AWS without this plugin. It's as simple as that. It enabled unicast discovery and S3 storage (for backups and recovery).

[Check it out at the Github repo](https://github.com/elastic/elasticsearch-cloud-aws)


Building the cluster using Cloudformation on AWS
-------------------
If you don't use AWS, feel free to skip this. However, if you do, this may decrease the time you spend creating the cluster - [using Cloudformation](https://aws.amazon.com/cloudformation/).
Cloudformation essentially automates the creation of all your AWS resources starting from roles, EC2 instances, security groups and much more.

By using CF, you can be up and running within seconds and recreating the cluster repeatedly isn't a tedious job. Some may argue how hard
is it to build a two-three node cluster, but there's always scope for human error and it's not exactly time well spent - especially if it can be automated.

### Cloudformation script for auto-scaling group

I created a fairly simple Cloudformation script that I've been re-using. It creates a multi-node auto-scaling group of ubuntu machines, security groups and ES role for the instances.

{% gist 65db44bf44507e74f6fe cloudformation-aws-elasticsearch.json %}

Please change the security groups based on what you need - at the moment they allow access from any IP. You may also need to tweak to rules so that instances have visibility of each other depending on your setup.

### Creating the stack
Using the provided script, you can upload it to AWS Cloudformation. Based on your script, you will need to fill in the parameters defined in the script.

<img alt="Cloudformation Parameters" src="/images/posts/ansible/cloudformation-options.png" class="" />

After that, AWS will go through each resources within the CF script and create it.

<img alt="Cloudformation Parameters" src="/images/posts/ansible/cloudformation-running.png" class="" />


Deploying Elastic Search
-------------------
Once your infrastructure is setup, you're going to need to actually deploy elastic search and all the plugins to the nodes. That's where ansible can help you out massively.
If you've never used ansible, it's a neat way to run scripts on remote machines. [Check it out here](http://docs.ansible.com/ansible/intro_getting_started.html)

There is ton of open source scripts that help you run different ansible roles.
The one I've been using is under [Traackr repo - ansible-elasticsearch](https://github.com/Traackr/ansible-elasticsearch). George Stathis, if you're ever in town, I owe you a beer!

It has support for pretty much all config you will need, will install plugins and will install the required java dependency.

Pull it down & get going git@github.com:Traackr/ansible-elasticsearch.git and add it as a role in your ansible scripts.

### Your Ansible script
Your mainl.yml script should look something alo the lines.

{{< highlight json "linenos=inline" >}}

- name: "Setup the Elastic Search on nodes."
  hosts: elasticsearchnodes
  remote_user: ubuntu
  roles:
    - { role: ansible-elasticsearch, sudo: yes }
  vars_files:
    - roles/ansible-elasticsearch/defaults/main.yml

{{< / highlight >}}

When you have added the ansible-elasticsearch as a folder in your roles, you should update your machines based on the boxes you created.

I also added cluster name and security group to the inventory vars below. You can also define specific node-names per instance.

{{< highlight bash "linenos=inline" >}}
[aws-es-01]
ec2-54-88-217-187.compute-1.amazonaws.com

[aws-es-01:vars]
elasticsearch_node_name=clever-node-name1

[aws-es-02]
ec2-54-164-77-248.compute-1.amazonaws.com

[elasticsearchnodes:children]
aws-es-01
aws-es-02

[elasticsearchnodes:vars]
elasticsearch_cluster_name=elasticsearch.cluster
elasticsearch_plugin_aws_ec2_groups=Elastic-Search-ElasticSearchSecurityGroup-1E0LR18NU18JG
{{< / highlight >}}

Of course, if you are running deployments on multiple machines this can be cumbersome. There are scripts that auto-generate inventories based on specific AWS tags - so you can deploy to all instances that have a tag that you've defined at creation time.

### Changing variables
You should then open *roles\ansible-elasticsearch\vars\main.yml* and edit variables accordingly.

Make sure you give ES enough HEAP_SIZE - the default 1GB is almost never enough for any use-case and pick your versions.
You can also specify any plugins you want to install along the way. The sample below includes both the kopf and AWS plugins mentioned above.

{{< highlight json "linenos=inline" >}}

elasticsearch_user: "ubuntu"
elasticsearch_group: "ubuntu"
elasticsearch_version: 1.7.2
elasticsearch_heap_size: 2g
elasticsearch_max_open_files: 65535
elasticsearch_timezone: "Europe/London"
elasticsearch_install_java: yes

elasticsearch_plugin_aws_version: 2.7.1
elasticsearch_plugins:
  - { name: 'lmenezes/elasticsearch-kopf/1.5.7' }

elasticsearch_service_startonboot: yes


{{< / highlight >}}


### Running the script
Finally, once you have your vars in place, run the main.yml ansible script (sample above)

{{< highlight bash "linenos=inline" >}}
ansible-playbook --private-key=[path-to-your-private-key] -i aws main.yml
{{< / highlight >}}

This will run the deployment on the machines specified in your inventory. At the end your cluster will be setup!

<img alt="Cloudformation Parameters" src="/images/posts/ansible/kopf-cluster.png" class="" />