---
layout: post
title: "Getting Started with ASP.NET Web API"
date: 2013-01-10 21:56:00 +0000
comments: true
thumbnail: /images/posts/webapi/blog-webapi_thumb.png
summary: "There are several options you can go for when deciding what technology you adopt for your RESTful API. In fact, you’ve got lots to chose from."
tags: [WebAPI, Asp.Net, MVC, C#]
---

<img src="/images/posts/webapi/blog-webapi_thumb.png" class="post-image-right" />
A significant part of ASP.NET MVC4 release was the Web API. In a nutshell, it’s a **powerful framework that makes creation of HTTP services easy and straight-forward**. In many ways, it’s something we’ve all been waiting for.

*The code sample for the article can be found on [**GitHub**](https://github.com/mirajavora/WebAPISample). You can find more details at the end of the article.*

Creating a RESTful API with ASP.NET
-------------------

There are several options you can go for when deciding what technology you adopt for your RESTful API. In fact, you’ve got lots to chose from. If you’re old-school (and enjoy living in the past), you might want to go for [**WCF**](http://msdn.microsoft.com/en-us/library/ms731082.aspx). Another popular open source framework for creating RESTful APIs with C# is [**OpenRasta**](http://openrasta.org/). A creation of Sebastien Lambda has been around for a while, however, it hasn’t had that much traction in the last 18 months.

Smaller frameworks such as [**NancyFx**](http://nancyfx.org/) or [**Simple.Web**](https://github.com/markrendle/Simple.Web) are also an option.

Why Should You Chose Web API Over Anything Else
-------------------

Obviously, ASP.NET Web API isn’t going to be fit for all. However, here are few reasons why you should consider Web API

### Simplicity, Convention Based, Designed with Rest in mind

Gives you a direct access to manipulate HTTP requests and responses and uses conventions to map HTTP methods to actions. It is simple and easy to get up and running, no XML-based setup required.

### Content Negotiation And Filters

It supports JSON and XML out of the box and lets you create your own formatters for whatever media type you specify through the content-type headers. It also provides filters that can be applied to each state of the action. It also comes with lots of goodies such as Model State validation.

### Routing

Web API uses the same mechanism to deliver resources as the rest of the ASP.NET stack – [routing](http://msdn.microsoft.com/en-GB/library/cc668201.aspx). It keeps the API endpoint implementations separated from the Uris. It also lets you use add constraints in the urls.

### IOC Support and Dependency Resolver

Comes with support for IOC out of the box. In fact, has been written so that almost every part (including the default content formatters) can be replaced.

Create a New Web API Endpoint
-------------------

In order to get started, you can either select **Web API Project** from Visual Studio create project dialog or you can install the [**Web API Nuget Package package**](https://nuget.org/packages/Microsoft.AspNet.WebApi/). This will pull down further dependencies such as (Microsoft.AspNet.WebApi.WebHost, Microsoft.AspNet.WebApi.Core, Microsoft.AspNet.WebApi.Client and Newtonsoft.Json).

Create your API endpoint by inheriting from [**ApiController**](http://msdn.microsoft.com/en-us/library/system.web.http.apicontroller(v=vs.108).aspx).

{{< highlight csharp "linenos=table" >}}
public class ContactController : ApiController
{
        public IEnumerable<Contact> Get()
        {
            ...
        }
 
        public Contact Get(Guid id)
        {
            ...
        }
 
        public HttpResponseMessage Post(Contact entity)
        {
            ...
        }
 
        public HttpResponseMessage Put(Guid id, Contact entity)
        {
            ...
        }
 
        public HttpResponseMessage Delete(Guid id)
        {
            ...
        }
}
{{< / highlight >}} 

The ApiController class uses conventions to map HTTP methods to actions. As long as the **action name starts with the HTTP method name**, it will be mapped correctly. For example, for HTTP DELETE method to remove Contact resource you should name the action DeleteContact or just Delete. However, if you don’t want to follow the pattern, you can also decorate each method with **HttpGet, HttpPut, HttpPost, or HttpDelete**.

The automatic content negotiation will then work out which formatter to use and present the same resource using the requested format. One thing to note is that **Web API uses Newtonsoft.Json as a default JSON serializer!**

HttpResponseMessage and HttpResponseException
-------------------

Web API gives you access to the raw HTTP response via the [HttpResponseMessage](http://msdn.microsoft.com/en-us/library/system.net.http.httpresponsemessage.aspx) and the [HttpRequestMessageExtensions](http://msdn.microsoft.com/en-us/library/system.net.http.httprequestmessageextensions.aspx). The extension class contains methods for creating responses beyond the simple 200 with a resource. You can access the extensions by calling **Request.ExtensionMethod**.

For example, when a PUT method is called, HTTP standard dictates that on a successful creation of the resource, the server should respond with 201 Created and Uri of the created resource in the Location header.

{{< highlight csharp "linenos=table" >}}
... validation and creation of the resource
 
HttpResponseMessage response = Request.CreateResponse<Contact>(HttpStatusCode.Created, resource);
response.Headers.Location = GetLocation(resource.Id);
return response;
{{< / highlight >}} 

In a similar fashion, at any point you can call the [HttpResponseException](http://msdn.microsoft.com/en-us/library/system.web.http.httpresponseexception.aspx) and encapsulate the HttpResponseMessage within it.

{{< highlight csharp "linenos=table" >}}
public Contact Get(Guid id)
{
    var entity = _repository.FindById(id);
    if (entity == null)
    {
        throw new HttpResponseException(new HttpResponseMessage(HttpStatusCode.NotFound));
    }
 
    return entity;
}
{{< / highlight >}} 

Model Validation and ModelState
-------------------

One of the awesome features of MVC is the model validation. Web API follows the same pattern and you can use the same annotation to use model validation in Web API. You can use the in-built validators or create your own custom validators. Calling **ModelState.IsValid** in your method will then evaluate the model.

{{< highlight csharp "linenos=table" >}}
public class Contact
{
    //...other fields
 
    [Required]
    public string Name { get; set; }
 
}
 
public HttpResponseMessage Post(Contact contact)
{
    if (ModelState.IsValid)
    {
        //.. handle successful creation of the contact resource
    }
    return Request.CreateResponse(HttpStatusCode.BadRequest);
}
{{< / highlight >}} 

Set Up Route To The API Endpoint
-------------------

Although the routing in Web API follows the same pattern as routing in MVC, it is important to understand they are not the same. You are calling MappHttpRoute on HttpRouteCollection with a routeTemplate.

{{< highlight csharp "linenos=table" >}}
public static void Register(HttpConfiguration config)
{
    config.Routes.MapHttpRoute(
        name: "DefaultApi",
        routeTemplate: "api/{controller}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );
}
{{< / highlight >}} 

If you want to read more on Web API routing, this article by [Mike Wasson](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api) is a good start.

Dependency Injection
-------------------

You can inject your Web API controllers with your own services using your own IDependencyResolver implementation. For more details, you can check out the code sample (Castle Windsor implementation). The only thing you need to do is set your own DependencyResolver implementation on App Start.

{{< highlight csharp "linenos=table" >}}
GlobalConfiguration.Configuration.DependencyResolver =
                     new WindsorDependencyResolver(Container);
{{< / highlight >}} 
 

Code Example
-------------------

You can check out all the the above in the [**code sample on GitHub**](https://github.com/mirajavora/WebAPISample). It covers a basic Contat Management API, Dependency Injection with Castle Windsor, Model Validation, Basic Routing setup and front-end based on Twitter Bootstrap and JQuery.