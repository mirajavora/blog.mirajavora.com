---
layout: post
title: "Octopress - An awesome blogging framework"
date: 2014-11-23 17:52:30 +0000
comments: true
thumbnail: /images/posts/octopress/logo.jpeg
summary: "I've been looking to re-launch my blog recently and be a bit more active in contributing to the community. While doing so, I wanted to change my existing blogging platform as well as hosting."
tags: [Blog, Octopress, Hosting]
archive: 2014
---

I've been looking to re-launch my blog recently and be a bit more active in contributing to the community. While doing so, I wanted to change my existing blogging platform as well as hosting.
<!--more-->

Looking For a Light-Weight Blogging Platform
-------------------

When I fist started blogging, I went for the [**Orchard Project**](http://www.orchardproject.net/) as the blogging platform. I was hoping that it would be light, MVC-based platform that would be easy to manage and maintain. I was a bit dissapointed with [Orchard](http://www.orchardproject.net/) - it was pretty heavy, development was cumbersome, the performance was low and overall felt like a bit of an overkill.

Looking at a new framework, I had several requirements. It had to be fast, simple and be able to handle

- posts
- archive & monthly archive
- widgets to load latest & custom
- ability to generate custom content pages

I also wanted move away from an IIS based hosting over to [**Nginx**](http://nginx.org/). That meant I had few open-source CMS options to look at. 
There is [Django CMS](https://www.django-cms.org/), [Joomla](http://www.joomla.org/) and of course [Drupal](https://www.drupal.org/).

And then I stumbled upon [Octopress](http://octopress.org/)

Octopress
-------------------

Unlike the traditional CMS projects, octopress is a blog-aware static content site generator. It runs on top of [Jekyll](https://github.com/jekyll/jekyll) which is the engine behind GitHub pages.

The content is generated from the markdown files you create. Jekyll alone does not have a great deal of html templates and css/js - that's where Octopress comes in.
It gives you an awesome framework that you can take forward and tweak to your liking, directly in the code.

The fact that all the **content is static makes your site super-fast** - as demonstrated in the graph below.

![Time to download the pages](/images/posts/octopress/download-time.png)

Living with Octopress
-------------------

Having used Octopress for few weeks now, the experience has been great. The ability to change html/css directly is great as well as no need to have data-store in place. The deployment is simple and straightforwards as well as backups. In short, exactly what I a was after.


### Installation

Installing Octopress is super-easy. The two main dependencies are Ruby and Git, which most devs will have anyway.

{{< highlight bash "linenos=inline" >}}
git clone git://github.com/imathis/octopress.git octopress
cd octopress
{{< / highlight >}}

You will then need to install couple of other gem dependencies and install octopress - to setup dirs and structure

{{< highlight bash "linenos=inline" >}}
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install

rake install
{{< / highlight >}}

Octopress has a concept of templates/layouts and includes which you can include to avoid repetition. You can find them in */source/_layouts* and */source/_includes*

### Writing Posts

Once you have Octopress up and running, you can start creating posts. There are rake methods to do that as a shortcut.

To create a new post, run

{{< highlight bash "linenos=inline" >}}
rake new_post["A new post Title"]
{{< / highlight >}}

This will create a new markdown file in */source/_posts*. Each post contains a set of variables and the actual content.
You can also add your own on top of the pre-defined.

{{< highlight html "linenos=inline" >}}
---
layout: post
title: "SASS with Visual Studio Part 1 Introduction"
date: 2011-09-11 15:01:30 +0000
comments: true
categories: [SASS, CSS, Visual Studio]
customVariable: "Custom content"
image: /images/posts/sass/sass_thumb.jpg
---

This is a start of the content ...
{{< / highlight >}}

Once you're happy with the content, you can run another rake task to produce the content. 

{{< highlight bash "linenos=inline" >}}
rake generate
{{< / highlight >}}

The generated files will end up in */public*

### Hacking Octopress

Octopress is an ideal choice for any developer. In fact, there was very little that it couldn't do out of the box or I couldn't change right away.

It was super easy to change any html and tweak any shared components. There are lots of [plugins available](https://github.com/imathis/octopress/wiki/3rd-party-plugins), the only additional bit that I was interested was a monthly archive which I borrowed from [here](https://github.com/rcmdnk/monthly-archive).


### Back ups

One of the nicest things about Octopress is the ease of back-ups. With no dependencies on DBs or any other storage, I tend to simply push the entire blog onto a [public github account](https://github.com/mirajavora/blog.mirajavora.com).

That means I have a permanent backup and I can clone and start updating the content anywhere. Changing hosting providers is a matter of minutes rather than hours.


### Deployment

All that octopress needs is an http server in front. It supports a simple push to something like GitHub pages or Rsync to your own server. You can also easily write your own rake tasks to push the content anywhere.


Any questions, give me a shout [@mirajavora](http://twitter.com/mirajavora)