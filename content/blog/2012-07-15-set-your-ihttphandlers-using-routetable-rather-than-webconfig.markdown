---
layout: post
title: "Set your IHttpHandlers using RouteTable rather than web.config"
date: 2012-07-15 09:46:00 +0000
comments: true
summary: "If for whatever reason you use IHttpHandlers in your MVC project, it may be that you are still using web.config to set the path to the handlers. This can get tricky and easy to miss, especially if you move from cassini dev to IIS7 live. There is however a better, neater way to declare paths to your handers."
tags: [C#, MVC, ASP.NET, Visual Studio]
archive: 2012
aliases:
    - /set-your-ihttphandlers-using-routetable-rather-than-webconfig
---

If for whatever reason you use IHttpHandlers in your MVC project, it may be that you are still using web.config to set the path to the handlers. This can get tricky and easy to miss, especially if you move from cassini dev to IIS7 live.

There is however a better, neater way to declare paths to your handers. If you are using MVC, you can register the handlers in RouteTable, along with your other routes. All you need to do is implement IRouteHandler interface and register a route using your custom route handler.
<!--more-->

Furthermore, you can add tokens on the route while you’re registering the handlers to the RouteCollection. These can be identified by the routehandler and passed down to the handler as arguments.

Lets assume you have a handler that looks after your file upload. The same handler is re-used for multiple different file upload processes and we need to be aware, which upload type it is. The handler should be registered at ~/handlers/uploadfile/{type}

First lets create the FileUploadHandlerRoute

{{< highlight csharp "linenos=inline" >}}
public class FileUploadHandlerRoute : IRouteHandler 
{
    public IHttpHandler GetHttpHandler(RequestContext requestContext)
    {
        var uploadType = ImageUploadType.Simple;
        var type = requestContext.RouteData.Values["type"].ToString();
        Enum.TryParse(type, out uploadType);
 
        return new FileUploadHandler(uploadType);
    }
}
{{< / highlight >}}

And then, simple add the handler route to your RouteCollection

{{< highlight csharp "linenos=inline" >}}
//..your other routes
 
routes.Add(
    "UploadFileHandler",
    new Route("handlers/uploadfile/{type}", new FileUploadHandlerRoute())
);
{{< / highlight >}}

If you like the post, have any questions or wish to shout some abuse follow me [@mirajavora](http://twitter.com/mirajavora)