---
layout: post
title: "SASS with Visual Studio Part 2 Features and Syntax"
date: 2011-09-11 23:00:00 +0000
comments: true
categories: [SASS, CSS, Visual Studio]
---

SASS is a super-set of CSS, that means, any existing CSS that you already wrote will just work. What we usually do, is paste any legacy CSS (if any) to our SASS file and take it from there. I’ve posted few examples of the syntax below. In my next post, I will focus on the real-world usage of SASS.
<!--more-->

The useful stuff – Nested Syntax
-------------------

If you like tidy CSS syntax, you’ll love this. Not my most favourite, but still worth mentioning. So something like this

{% highlight css %}
    ul
    {
        margin: 0;
        li 
        {
            float: left;
        }
    }
{% endhighlight %}

will be transferred into

{% highlight css %}
    ul { margin: 0; }
    ul li {float: left;}
{% endhighlight %}

 
Cool Stuff – Variables
-------------------

SASS lets you create variables and re-use them throughout. These may be plain your colours that you re-use or size values. Using SASS syntax, you can perform various calculations on them if need be.

Declaring SASS variables




results into



This will pull down and install the compass gem and all the dependencies.

Setup

Once you have pulled down the gem, you can call compass to set-up your project. This will set-up the initial SCSS source files and folder structure.

You can either chose to go for the default setup

{% highlight ruby %}
compass create Content
{% endhighlight %}

or you can try add params to customise the setup

{% highlight ruby %}
$ compass create Content --sass-dir "src" --css-dir "css" --javascripts-
dir "Javascript"
{% endhighlight %}
 

This will create the directory structure, config file, adds a default blue-prints and few examples. The example uses “Content” as the project folder, where all CSS, JS, SCSS and images are located.

If you prefer to create just the config file with the folder structure, add the - -bare param

{% highlight ruby %}
$ compass create Content --bare --sass-dir "src" --css-dir "css" --javascripts-
dir "Javascript
{% endhighlight %}


{% highlight ruby %}
[rectangle setX: 10 y: 10 width: 20 height: 20];
{% endhighlight %}
  
Watch the folder for changes

The last thing you need to do, is to watch the SASS folder for changes. You can manually run compass to translate the SCSS files to CSS. However, it’s more elegant to have compass watch the folder and automatically create the CSS file when a change is detected. You can also run the command during your build process with different setup (compression, etc).

You can either call the watch command from your console

![Console Watch](/images/posts/sass/blog-compass-watch_thumb.png "Console")

What we do is have a bat file in the watch command in the root of every project.

