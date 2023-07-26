---
layout: post
title: "SASS with Visual Studio Part 2 Features and Syntax"
date: 2011-09-11 23:00:00 +0000
comments: true
summary: "SASS is a super-set of CSS, that means, any existing CSS that you already wrote will just work. What we usually do, is paste any legacy CSS (if any) to our SASS file and take it from there. I’ve posted few examples of the syntax below. In my next post, I will focus on the real-world usage of SASS."
tags: [SASS, CSS, Visual Studio]
---

SASS is a super-set of CSS, that means, any existing CSS that you already wrote will just work. What we usually do, is paste any legacy CSS (if any) to our SASS file and take it from there. I’ve posted few examples of the syntax below. In my next post, I will focus on the real-world usage of SASS.
<!--more-->

The useful stuff – Nested Syntax
-------------------

If you like tidy CSS syntax, you’ll love this. Not my most favourite, but still worth mentioning. So something like this

{{< highlight css "linenos=inline" >}}
    ul
    {
        margin: 0;
        li 
        {
            float: left;
        }
    }
{{< / highlight >}}

will be transferred into

{{< highlight css "linenos=inline" >}}
    ul { margin: 0; }
    ul li {float: left;}
{{< / highlight >}}

 
Cool Stuff – Variables
-------------------

SASS lets you create variables and re-use them throughout. These may be plain your colours that you re-use or size values. Using SASS syntax, you can perform various calculations on them if need be.

Declaring SASS variables

{{< highlight css "linenos=inline" >}}
    $primary-color: #ccc;
    $secondary-color: #ebebeb;
    $grid-height: 20;
     
    a { color: $primary-color;}
    a:hover { color: $secondary-color;}
    div.test { height: $grid-height + 10 + px;}
{{< / highlight >}}

results into

{{< highlight css "linenos=inline" >}}
    a {
      color: #cccccc;
    }
    a:hover {
      color: #ebebeb;
    }
    div.test {
      height: 30px;
    }
{{< / highlight >}}


Really Cool Stuff- Mixins and Inheritance
-------------------

One of the key features of sass is the ability to declare mixins. These are basically functions that you can declare to run common tasks.  For example, we can declare a function that handles background images for us

{{< highlight css "linenos=inline" >}}
    @mixin background($url, $color:transparent, $repeat:no-repeat, $position:0 0) {
        background: $color url($url) $repeat $position;
    }
     
    div {
        @include background('/some/url');
    }
     
    div.second {
        @include background('/some/url', '#ebebeb');
    }
{{< / highlight >}}

which will transform into

{{< highlight css "linenos=inline" >}}
    div {
      background: transparent url("/some/url") no-repeat 0 0;
    }
    div.second {
      background: "#ebebeb" url("/some/url") no-repeat 0 0;
    }
{{< / highlight >}}

The Really, Really Cool Stuff – Compass Libraries
-------------------

Now since we use compass, we can also tap into some existing methods and libraries inside compass. The compass core includes reset, layout and typography helpers, CSS helpers and CSS3 libs and utilities.

Check them out at [http://compass-style.org/reference/compass/](http://compass-style.org/reference/compass/)

Uber Cool Stuff – Maths, Operations, Loops and conditions
-------------------

Bring it all together, and you can go wild … quite wild … something like this

{{< highlight css "linenos=inline" >}}
    $gutterWidth: 20;
    $columns: 12
    $widthForColumns: $totalWidth - ($columns * $gutterWidth);
    $columnWidth: floor($widthForColumns/$columns);
     
    @for $i from 1 through $columns {
      .container_12 .grid_#{$i} { width:(($columnWidth*$i) + (($gutterWidth*$i)-$gutterWidth))+px; }
    }
{{< / highlight >}}

Related Articles
-------------------

[SASS with Visual Studio Part 1 Introduction](/introduction-to-sass-with-visual-studio/)<br/>
[SASS with Visual Studio Part 2 Features and Syntax](/sass-with-visual-studio-part-2-features-and-syntax/)<br/>
[SASS with Visual Studio Part 3 Real World](/sass-with-visual-studio-part-3-real-world/)<br/>
[SASS/SCSS and Compass Part 4 Spriting](/sass-part-4-spriting/)<br/>
