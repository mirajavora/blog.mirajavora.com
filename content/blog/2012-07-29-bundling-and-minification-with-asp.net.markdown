---
layout: post
title: "Bundling and Minification with ASP.NET 4.5"
date: 2012-07-29 15:33:00 +0000
comments: true
summary: "Minimising the number of requests the page has to perform can have a considerable effect on your site’s performance. IE6 and IE7 both limit the number of concurrent requests to 2, IE8 can handle up to 6. There is a lot you can to improve the initial load speed speed – one of which is bundling all your CSS and JS into two separate files. How much of a difference it could do. Well, as it turns out up to 30seconds on slower connections."
tags: [C#, MVC, Bundling, CSS, Javascript, Visual Studio]
---


Minimising the number of requests the page has to perform can have a considerable effect on your site’s performance. IE6 and IE7 both limit the number of concurrent requests to 2, IE8 can handle up to 6. There is a lot you can to improve the initial load speed speed – one of which is bundling all your CSS and JS into two separate files. How much of a difference it could do. Well, as it turns out up to 30seconds on slower connections.
<!--more-->

Bundled and Minified vs. Non-Bundled
-------------------

This is the standard ASP.NET MVC4 app with all the initial JS libraries and CSS. Over a slower connection, the difference can be up to 30seconds. However, even on faster connections, you can save up two seconds just by combining and minifying your scripts.

<div class="row-fluid">
    <a href="/images/posts/bundling/non-bundled_4.png"><img src="/images/posts/bundling/non-bundled_thumb_1.png" alt="Non Bundled" style="float:left" /></a>
    <a href="/images/posts/bundling/bundled_2.png"><img src="/images/posts/bundling/bundled_thumb.png" alt="Non Bundled" style="float:left" /></a>
</div>

Bundling and Minification in ASP.NET 4.5
-------------------

Luckily for us, the ASP.NET now ships with a new library called System.Web.Optimization. It provides pluggable bundling and minification functionality for your scripts and styles.

It lets you define bundles at application start and pass them to the BundleCollection. Creating a basic new bundle is quite simple. Let’s assume we would like to combine few CSS files

{{< highlight csharp "linenos=table" >}}
protected void Application_Start()
{
    ... other startup logic
 
    var cssBundle = new StyleBundle("~/Content/themes/base/css")
            .Include("~/Content/themes/base/jquery.ui.core.css",
            "~/Content/themes/base/jquery.ui.resizable.css");    
    BundleTable.Bundles.Add(cssBundle);
}
{{< / highlight >}}


You can also create bundles for your JavaScript.

{{< highlight csharp "linenos=table" >}}
protected void Application_Start()
{
    ... other startup logic
 
    var jsValidationBundle = new ScriptBundle("~/bundles/jqueryval")
              .Include("~/Scripts/jquery.unobtrusive*",
                        "~/Scripts/jquery.validate*"));
    BundleTable.Bundles.Add(jsValidationBundle);
}
{{< / highlight >}}

Both StyleBundle and ScriptBundle take url of the bundled file as a constructor argument and use extension method .Include to add files. You can also use wildcard characters such as * in the include array. If you want to add the entire folder, use IncludeDirectory extension.

One thing to note, is what version of the System.Web.Optimization you have. The older version that came with the MVC4 beta used AddFile() syntax to add files to the bundles. However, if you install VS 2012RC you get a newer version of the DLL, which is a bit neater and uses the syntax shown above.

Rendering Helpers
-------------------

The library also comes with two awesome helpers. When you develop locally, you want to have all the bundling setup, but you don’t want the bundling and minification to happen – it’s much easier to debug. The System.Web.Optimization has two helpers that address exactly that issue.

{{< highlight csharp "linenos=table" >}}
    <head>
        ... other content
        @Styles.Render("~/Content/themes/base/css", "~/Content/css")
        @Scripts.Render("~/bundles/jqueryval")
    </head>
{{< / highlight >}}
    
When you run the debug setting in the compilation element in your web.config as false, the Styles and Scripts helpers will render the bundled files. However, settings debug=”true” will render them unbundled. Pretty cool!

{{< highlight xml "linenos=table" >}}
<system.web>
    .....
    <compilation debug="false" targetFramework="4.5" />
    ....
</system.web>
{{< / highlight >}}

And that’s not everything, the minified files will also have cache busting string attached based on the files in the bundles.

{{< highlight html "linenos=table" >}}
<link href="/Content/themes/base/css?v=UM624qf1uFt8dYtiIV9PCmYhsyeewBIwY4Ob0i8OdW81" rel="stylesheet" type="text/css" />
{{< / highlight >}}