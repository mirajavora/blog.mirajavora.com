---
layout: post
title: "SASS with Visual Studio Part 3 Real World"
date: 2011-10-17 21:19:00 +0000
comments: true
categories: [SASS, CSS, Visual Studio]
---

Since I already covered the intro, install, features and syntax of SASS. It’s time to show some real-world application of SASS. If used properly, SASS can really save you time.
<!--more-->

Sprites
-------------------

One area that I find SASS particularly helpful is sprite creation. The idea optimising your site by pasting few images together and decreasing the number of requests is not new and is pretty cool. Usually, you would create the image and then manually write the CSS or upload the files on-by-one to online services that would create the spites for you. With SASS, you have can create a helper method … and that’s it!

{% highlight css %}
@mixin sprite($image, $numberOfItems) {
    $itemHeight: image-height($image) / $numberOfItems;
    $height: 0;
 
    #nav li a {
        background-image: image-url($image);
    }
 
    @for $i from 1 through $numberOfItems {
         
        #nav li a.item#{$i} {
            background-position:0px $height;
        }
        #nav li a:hover.item#{$i} {
            background-position:100px $height;
        }
        $height: $itemHeight + $height;
    }    
}
 
 
@include sprite("image.jpeg", 2);
{% endhighlight %}

This mixin will let you easily create any sprites and result in CSS below. Note the usage of image-url and image-height methods. They are in-build compass features, that will automatically get properties form the set image.

{% highlight css %}
#nav li a {
  background-image: url('/images/image.jpeg?1308598897');
}
#nav li a.item1 {
  background-position: 0px 0;
}
#nav li a:hover.item1 {
  background-position: 100px 0;
}
#nav li a.item2 {
  background-position: 0px 968px;
}
#nav li a:hover.item2 {
  background-position: 100px 968px;
}
{% endhighlight %}
 

If you want to go even further, compass will create sprites for you automatically. Check out sprite utils.

 

Mixing CSS
-------------------

Another cool way to use SASS is the ability to combine multiple scss files together. In this way, you can mix and match between your stylesheets, move your mixins and variables into separate reusable files and then decide what you need. SASS will pull it all together into single file.

{% highlight css %}
@import "_base";
@import "_reset";
@import "_mixins";
@import "_iefixes";
 
/* scss here */
{% endhighlight %}
 

Imports all the scss files into a single file – great for re-using parts.

Grid Systems
-------------------

We push 3-4 apps live every month, most of them based on the grid system. However, some of them would have 2-3 different layouts – a site based on 960px, Facebook tab on 520px and canvas on 760px. Each of them also have different gutters, different column numbers, sometimes some other funk in them. In short, we deal with loads of grid variations. There are sites that create these for you, but it’s faster if you have something that does it all for you that you can alter in place.

Just configure the

{% highlight css %}
$totalWidth: 520;
$gutterWidth: 20;
$columns: 12;
{% endhighlight %}

and you SASS will re-create the grid for you.

Related Articles
-------------------

[SASS with Visual Studio Part 1 Introduction](/introduction-to-sass-with-visual-studio/)<br/>
[SASS with Visual Studio Part 2 Features and Syntax](/sass-with-visual-studio-part-2-features-and-syntax/)<br/>
[SASS with Visual Studio Part 3 Real World](/sass-with-visual-studio-part-3-real-world/)<br/>
[SASS/SCSS and Compass Part 4 Spriting](/sass-part-4-spriting/)<br/>
