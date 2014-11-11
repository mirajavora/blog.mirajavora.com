---
layout: post
title: "SASS with Visual Studio Part 1 Introduction"
date: 2011-09-11 15:01:30 +0000
comments: true
categories: [SASS, CSS, Visual Studio]
image: /images/posts/sass/sass_thumb.jpg
---

Syntactically Awesome Stylesheets (SASS) aims to make writing CSS easy, re-usable and less repetitive.  The new SCSS (Sassy CSS) syntax makes use of variables, mixins, nested rules and inheritance to achieve this goal. Furthermore, I will show you how to use compass to leverage in-built functionality. This series of posts will not argue between SASS and LESS, it will be a quick guide on how to get up and running with SASS within VS and how SASS can help you.
<!--more-->

Install Ruby
-------------------

The first thing to do is to install ruby. The quickest and easiest is to run ruby installer from [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)

![Ruby Installer](/images/posts/sass/ruby-install_thumb.png "Ruby Installer")

 
Get Compass Gem
-------------------

Next, get the compass gem. Compass provides a framework that will translate the SCSS code into CSS. However, it also contains a bunch of other libraries that you can then make full use of. Most notably mixins around CSS3 functionality, reset, layouts, but also cool helpers for images, prefixes, files urls and loads more.

To get the gem, open up the command prompt and type in

{% highlight ruby %}
gem install compass
{% endhighlight %}

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

Related Articles
-------------------

[SASS with Visual Studio Part 1 Introduction](/introduction-to-sass-with-visual-studio/)<br/>
[SASS with Visual Studio Part 2 Features and Syntax](/sass-with-visual-studio-part-2-features-and-syntax/)<br/>
[SASS with Visual Studio Part 3 Real World](/sass-with-visual-studio-part-3-real-world/)<br/>

