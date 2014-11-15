---
layout: post
title: "Setting Properties In Models With Strongly Typed Model Filters ASP.Net MVC C#"
date: 2011-11-15 18:53:00 +0000
comments: true
summary: "I like strongly typed view models in MVC. Typical structure of my site in Razor View Engine includes a page base model that is used in the Layout template and then I use inheritance to add properties to each model or commonly re-used properties. There are various reasons why to avoid dynamic in your models, besides the fact that you cannot use dynamic in master frame using razor."
categories: [C#, MVC, Visual Studio]
---

I like strongly typed view models in MVC. Typical structure of my site in Razor View Engine includes a page base model that is used in the Layout template and then I use inheritance to add properties to each model or commonly re-used properties. There are various reasons why to avoid dynamic in your models, besides the fact that you cannot use dynamic in master frame using razor.
<!--more-->

The problem with model properties instead of something like ViewBag is that your controllers and services can start to grow with logic that shouldn’t really be there.

There are various ways to deal with this, one of them is assigning the properties you want commonly in your view models using filters.

To make the life a bit easier. I wrote ModelFilterBase<T> that looks for a specific model on result execution and triggers OnModelLoaded method, if the model is found.

Real world example
-------------------

Lets say you want every view model to contain information about the browser that is used. We can create an interface and implement the interface on the view model.

{% highlight c# %}
public interface IBrowserDetails
{
    string BrowserName {get;set;}
}
 
//implement the interface on the model or model base
public class HomeModel : IBrowserDetails
{
    public string BrowserName {get;set;}
    //... other properties
}
{% endhighlight %}

Then, implement the BrowserModelFilter with IBrowserDetails interface as type. Override the OnModelLoaded method and notice the protected model property with IBrowserDetails as type. Also, don’t forget to register the filter.

{% highlight c# %}
public class PermissionModelFilter : PresentationModelFilterBase<IBrowserDetails>
{
    protected override void OnModelLoaded(ResultExecutingContext filterContext)
    {
        var capabilities = filterContext.HttpContext.Request.Browser;
        Model.BrowserName = capabilities.Browser;
    }
}
{% endhighlight %}

The example above is a trivial. However, you can add complex logic or inject services into your filters.

The PresentationModelFilterBase Class
-------------------

The implementation of the model filter base class:

{% highlight c# %}
public abstract class PresentationModelFilterBase<T> : ActionFilterAttribute where T : class
    {
        protected T Model;
 
        public override void OnResultExecuting(ResultExecutingContext filterContext)
        {
            base.OnResultExecuting(filterContext);
 
            Model = filterContext.Controller.ViewData.Model as T;
            if (Model == null) return;
 
            OnModelLoaded(filterContext);
        }
 
        protected abstract void OnModelLoaded(ResultExecutingContext filterContext);
     }
{% endhighlight %}

You can download the whole sample below.

<iframe style="padding-bottom: 0px; background-color: #fcfcfc; padding-left: 0px; width: 98px; padding-right: 0px; height: 115px; padding-top: 0px" title="Preview" marginheight="0" src="https://skydrive.live.com/embedicon.aspx/Blog/ModelFilterBase.zip?cid=84e23a97f665c5f2&amp;sc=documents" frameborder="0" marginwidth="0" scrolling="no"></iframe>