---
layout: post
title: "Deep Dive into ASP.NET Bundling and Minification"
date: 2012-08-21 21:14:00 +0000
comments: true
summary: "The new **System.Web.WebPages 2.0.0.0** assembly that ships with the latest MVC4 contains a pretty cool feature that lets you override the current browser capabilities. Sure, most modern browsers let you set a custom user agent string out of the box or via extensions. However, there are certain scenarios, where you would want to switch the user agent on the server side. That’s where the **BrowserHelpers** class comes in handy."
tags: [C#, MVC, Asp.Net, Visual Studio]
---

The new **System.Web.WebPages 2.0.0.0** assembly that ships with the latest MVC4 contains a pretty cool feature that lets you override the current browser capabilities. Sure, most modern browsers let you set a custom user agent string out of the box or via extensions. However, there are certain scenarios, where you would want to switch the user agent on the server side. That’s where the **BrowserHelpers** class comes in handy.
<!--more-->

Override Browser Context
-------------------

A good example where you want to use override the browser capabilities is when developing mobile views. You may not want to simulate a particular device, you just want to tell ASP.NET that the client is a mobile device and to use the .mobile view.  You can call SetOverridenBrowser extension method and pass in BrowserOverride enum (Mobile/Desktop options).

{{< highlight csharp "linenos=inline" >}}
public ActionResult Mobile()
{
    HttpContext.SetOverriddenBrowser(BrowserOverride.Mobile);
    return RedirectToAction("Index");
}
{{< / highlight >}}

If you want, you can override the browser full UserAgent by calling SetOverridenBrowser extension method on HttpContextBase

{{< highlight csharp "linenos=inline" >}}
public ActionResult Iphone()
{
    HttpContext.SetOverriddenBrowser("Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293 Safari/6531.22.7");
    return RedirectToAction("Index");
}
{{< / highlight >}}

And then, in order the clear the override, simple call the ClearOverridenBrowser extension method

{{< highlight csharp "linenos=inline" >}}
public ActionResult Clear()
{
    HttpContext.ClearOverriddenBrowser();
    return RedirectToAction("Index");
}
{{< / highlight >}}
 

What is happening under the hood
-------------------

When you call the SetOverridenBrower method, ASP.NET sets a “.ASPXBrowserOverride” cookie. This is done using CookieBrowserOverrideStore from System.Web.Webpages, which implements BrowserOverrideStore – if you’re interested, check it out in dotpeek.

The value of the cookie is the user agent that you have set or in the case of the BrowserOverride.Mobile enum: Mozilla/4.0 (compatible; MSIE 6.0; Windows CE; IEMobile 8.12; MSIEMobile 6.0). The expiry date is set for 7 days so the override will be in place even if you re-open your browser. Calling ClearOverridenBrowser simply clears the cookie.

Create Mobile Switched Filter
-------------------

The jQuery.Mobile.MVC package comes with the ViewSwitcher razor partial and the ViewSwitcherController. This does more or less exactly what I described above. However, if you are lazy like me, you may want to switch between mobile/desktop views using QueryString rather than controller/actions.  This is useful when you want to just quickly check your mobile views.

{{< highlight csharp "linenos=inline" >}}
public class BrowserCapabilitiesSwitcherFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        var switchParameter = filterContext.RequestContext.HttpContext.Request.QueryString["switch"];
        if(string.IsNullOrEmpty(switchParameter))
            return;
 
        var browserOverride = BrowserOverride.Desktop;
        if(Enum.TryParse(switchParameter, true, out browserOverride))
        {
            //switch between BrowserOverride.Desktop / BrowserOverride.Mobile
            filterContext.RequestContext.HttpContext.SetOverriddenBrowser(browserOverride);
        }
        else
        {
            //set the user-agent string
            filterContext.RequestContext.HttpContext.SetOverriddenBrowser(switchParameter);
        }            
    }
}
{{< / highlight >}}

Simply use it by typing http://yoursite.com/page?switch=Mobile to preview in mobile and then http://yoursite.com/page?switch=Desktop to switch back. For the more adventurous, you can pass in the user agent directly http://yoursite.com/page?switch=UserAgentString

Any questions or comments, tweet me [@mirajavora](http://twitter.com/mirajavora)