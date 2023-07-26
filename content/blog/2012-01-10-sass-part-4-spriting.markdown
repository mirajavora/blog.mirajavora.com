---
layout: post
title: "SASS/SCSS and Compass Part 4 Spriting"
date: 2012-01-10 22:34:00 +0000
comments: true
summary: "Statistically Awesome Style Sheets with Compass can help you easily generate sprites. Not only that, it will re-generate the images, re-calculate the position and update the CSS every time you update the source images. This post explains the basics and aims to address the most common scenarios when creating sprites."
tags: [C#, Castle Windsor, ASP.NET]
---

Statistically Awesome Style Sheets with Compass can help you easily generate sprites. Not only that, it will re-generate the images, re-calculate the position and update the CSS every time you update the source images. This post explains the basics and aims to address the most common scenarios when creating sprites.
<!--more-->

What Are Sprites
-------------------

Sprites exist to reduce the number of requests to the server by combining multiple images in one. This can make a significant performance and usability improvement to your site. Using CSS you can show/hide areas of the large sprite image.

How does it work
-------------------

To create a sprite with SASS, simply place all your images that should be part of the sprite in a folder in your images folder. Compass will generate a sprite file and will place it on the same level as the folder. IE if the folder was called /content/images/icons/, it will generate /content/images/icons-{hash}.png file.

You need to have compass watch command running and your images folder can be defined in compass.rb file. If you’re not sure how to get these up and running, check out my introduction to SASS post.

Simple Rollover Image Button
-------------------

One of the most common sprites – rollover image. First, place the two image states to the folder called submit (normal.png, hover.png). IE /Content/images/submit/normal.png  and /Content/images/submit/hover.png

First declare the mixin and import compass lib for spriting

{{< highlight ruby "linenos=table" >}}
@import "compass/utilities/sprites/base";
@mixin spriteHoverButton($selector, $spriteMap) {
    #{$selector} {
    float: left;
    overflow: hidden;
    padding: 0;
    text-indent: -1000px;
    background: sprite($spriteMap, normal) no-repeat;
    @include sprite-dimensions($spriteMap, normal);
    }
 
    #{$selector}:hover {
        background-position: sprite-position($spriteMap, hover);
    }
}
{{< / highlight >}}

Then define the sprite map from the images (this will create the sprite)

{{< highlight ruby "linenos=table" >}}
$continue: sprite-map("submit/*.png");
{{< / highlight >}}

You will notice that submit-{hash}.png was created. Finally, use the mixin to create the CSS button with specified selector

{{< highlight ruby "linenos=table" >}}
@include spriteHoverButton('a.continue', $continue);
{{< / highlight >}}

Remember, every time you update the images in the submit folder, compass will automatically re-generate the sprite and update the dimensions.

Generated CSS

{{< highlight css "linenos=table" >}}
/** Submit Sprite **/
a.continue {
  float: left;
  overflow: hidden;
  padding: 0;
  text-indent: -1000px;
  background: url('/images/submit-sb839570bbd.png') 0 -37px no-repeat;
  height: 37px;
  width: 137px;
}
 
a.continue:hover {
  background-position: 0 0;
}
{{< / highlight >}}

Creating Icons Sprite
-------------------

Alternatively, instead of defining the sprite map and getting properties using the file names ie sprite-position($spriteMap, hover), you can let compass and SASS handle everything for you. This is particularly handy when creating large icon sprites where you worry less about additional CSS syntax.

Simply place all the images in the single directory. EG /images/icons/one.png /images/icons/two.png …..  Then, import the compass spriting base and the icons folder

{{< highlight ruby "linenos=table" >}}
@import "compass/utilities/sprites/base";
@import "icons/*.png";
{{< / highlight >}}

And finally call all-{foldername}-sprites mixin. Make sure you use the correct foldername, IE if folder name was icons, then the call would be all-icons-sprites.

{{< highlight ruby "linenos=table" >}}
@include all-icons-sprites;
{{< / highlight >}}

This will go through every single png file in the folder and create sprite markup using the image name.  Resulting in following CSS

{{< highlight ruby "linenos=table" >}}
.icons-sprite, .icons-four, .icons-one, .icons-three, .icons-two {
  background: url('/images/icons-s337020fd3d.png') no-repeat;
}
 
.icons-four {
  background-position: 0 0;
}
 
.icons-one {
  background-position: 0 -37px;
}
 
.icons-three {
  background-position: 0 -74px;
}
 
.icons-two {
  background-position: 0 -111px;
}
{{< / highlight >}}

Have fun! If you like to know more or are just bored, follow me on @mirajavora :-)

Related Articles
-------------------

[SASS with Visual Studio Part 1 Introduction](/introduction-to-sass-with-visual-studio/)<br/>
[SASS with Visual Studio Part 2 Features and Syntax](/sass-with-visual-studio-part-2-features-and-syntax/)<br/>
[SASS with Visual Studio Part 3 Real World](/sass-with-visual-studio-part-3-real-world/)<br/>
[SASS/SCSS and Compass Part 4 Spriting](/sass-part-4-spriting/)<br/>
