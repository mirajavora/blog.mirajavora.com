---
layout: post
title: "Deep Dive into ASP.NET Bundling and Minification"
date: 2012-08-04 21:50:00 +0000
comments: true
summary: "In the previous post, I went on about how to use System.Web.Optimization library to minimize your page load times. However, the new library offers quite a lot of extensibility and even if you don’t want to use the default minification, you can still use the framework."
tags: [C#, MVC, Bundling, CSS, Javascript, Visual Studio]
archive: 2012
---

In the previous post, I went on about how to use System.Web.Optimization library to minimize your page load times. However, the new library offers quite a lot of extensibility and even if you don’t want to use the default minification, you can still use the framework.
<!--more-->

Bundle Without Minification
-------------------

Sometimes, first step is just to bundle the files together without minification. I often find some JS is poorly written and is missing the odd semi-colon, which blows up during minificaiton.

Unfortunately, there is not a quick flag you can set on the bundle. The good news is, it’s fairly simple. You can create your own transform and pass it to the bundle. Remember to use Bundle instead of StyleBundle or ScriptBundle.

{{< highlight csharp "linenos=inline" >}}
public class NonMinifying : IBundleTransform
{
    private readonly string _contentType;

    public NonMinifying(string contentType = "text/css")
    {
        _contentType = contentType;
    }

    public void Process(BundleContext context, BundleResponse bundle)
    {
        if (bundle == null)
        {
            throw new ArgumentNullException("bundle");
        }

        context.HttpContext.Response.Cache.SetLastModifiedFromFileDependencies();
        foreach (FileInfo file in bundle.Files)
        {
            HttpContext.Current.Response.AddFileDependency(file.FullName);
        }

        bundle.ContentType = _contentType;
    }
}
{{< / highlight >}}


{{< highlight csharp "linenos=inline" >}}
//... in your app init
IBundleTransform cssTramsform = new NonMinifying();

var cssBundle = new Bundle("~/Content/themes/base/css", cssTramsform)
            .Include("~/Content/themes/base/jquery.ui.core.css",
            "~/Content/themes/base/jquery.ui.resizable.css");
    BundleTable.Bundles.Add(cssBundle);
{{< / highlight >}}


Plugin Your Own Minification
-------------------

If you don’t like the default minification that comes in with the System.Web.Optimization, you can plug in your own. I’ve used SquishIt few times – lets have a look have we can integrate it in the ASP.NET bundles.

All we need to do is write our own custom SquishIt transform. It will iterate through each file in the bundle and process it.

{{< highlight csharp "linenos=inline" >}}
public class SquishItCssTransform<T> : IBundleTransform where T : IMinifier<CSSBundle>
{
    public void Process(BundleContext context, BundleResponse response)
    {
        var cssCompressor = MinifierFactory.Get<CSSBundle, T>();
        var rawCss = new StringBuilder();

        foreach (var fileInfo in response.Files)
        {
            using(var reader = fileInfo.OpenText())
            {
                rawCss.Append(reader.ReadToEnd());
            }
        }
        var compressedCss = cssCompressor.Minify(rawCss.ToString());
        context.HttpContext.Response.Cache.SetLastModifiedFromFileDependencies();
        response.Content = compressedCss;
        response.ContentType = "text/css";
    }
}
{{< / highlight >}}


{{< highlight csharp "linenos=inline" >}}
....
//in your bundling init
var cssTransform = new SquishItCssTransform<YuiMinifier>();
BundleTable.Bundles.Add(new Bundle("~/Content/css", cssTransform)
.Include("~/Content/site.css"));
{{< / highlight >}}

The SquishItCssTransform takes type of T so you can specify the type of the CSS minifier from the SquishIt library. In this example, I’ve used the YuiMinifier.


CoffeeScript or SASS, no problem!
-------------------

If you use CoffeeScript for your JavaScript or SASS/LESS for your CSS, the process is pretty much similar. You need to create a new bundle and pass in your custom IBundleTransform. Using CoffeeScript compiler is dead easy.

{{< highlight csharp "linenos=inline" >}}
public class JavascriptCoffeeScriptTransform : IBundleTransform
{
    public void Process(BundleContext context, BundleResponse response)
    {
        var compiler = new CoffeeScriptCompiler();
        var content = new StringBuilder();
        foreach (var fileInfo in response.Files)
        {
            using (var reader = fileInfo.OpenText())
            {
                content.Append(reader.ReadToEnd());
            }
        }

        var compiled = compiler.Compile(content.ToString());
        var factory = MinifierFactory.Get<JavaScriptBundle, JsMinMinifier>();

        var minified = factory.Minify(compiled);

        context.HttpContext.Response.Cache.SetLastModifiedFromFileDependencies();
        response.Content = minified;
        response.ContentType = "text/javascript";
    }
}
{{< / highlight >}}


{{< highlight csharp "linenos=inline" >}}
//.. your bundling setup ...
var coffeeBundleAndMinify = new JavascriptCoffeeScriptTransform();
var coffeeBundle = new Bundle("~/bundles/coffee", coffeeBundleAndMinify)
                       .Include("~/Scripts/Coffee/Script.coffee");
BundleTable.Bundles.Add(coffeeBundle);
{{< / highlight >}}


Now you have a bundle from CoffeeScript files that is compiled and minified.

Related
-------------------

[Bundling and Minification with ASP.NET MVC 4](/bundling-and-minification-with-asp.net)

Any questions, tweet me [@mirajavora](http://twitter.com/mirajavora)