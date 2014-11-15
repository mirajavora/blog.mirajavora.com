---
layout: post
title: "Render Email Templates Using Razor Engine"
date: 2012-10-08 08:22:00 +0000
comments: true
image: /images/posts/razor/EmailTemplate_thumb.png
summary: "A larger web projects would typically consist not only of front end web project, but would include additional class libraries and offload some of the heavy processing work to service or console apps. The common problem is then how do you update the front-end and signal the site that some work has been completed."
categories: [C#, Razor, Asp.Net, Visual Studio]
---

Sooner of later, there comes a point you will be asked to implement email templates. Tools such as Campaign Monitor are useful, but they are not cost-effective when you reach a larger number of users and you have in-house skills to create an email system.

The Old School – XML And XSLT or Token Replacement
-------------------

A common way to implement email templates was to use XML in combination with XSLT. It used to make sense: you separate the data from the markup, are able to create multiple XSLT tranforms based on HTML vs PlainText using the same data, you could use complex logic and loops and you could re-use the templates cross-systems. However, editing and maintaining the templates was never easy. Simpler systems may even use token replacement techniques.

Razor Engine To Generate Your Email Templates
-------------------

Razor engine became the new default view engine for ASP.NET MVC projects. In fact, there are at least [4 major view engines](http://stackoverflow.com/questions/1451319/asp-net-mvc-view-engine-comparison) and lots of others to choose from. A view engine takes a model and generates HTML. However, a view engine is also a perfect tool for creating email templates.

[RazorEngine project](http://razorengine.codeplex.com/) is built on Microsoft’s Razor parsing technology. The key thing is that you **don’t need web-context dependencies to parse the templates** or any other related razor dependencies.

### Why You Should Consider Razor Engine For Your Email Templates

Out of the box you get:EmailTemplate

- Ability to use strongly typed or anonymous models
- Conditional statements
- Loops 
- Clean Razor syntax and IntelliSense

On top of that, if you use Razor as your view engine in MVC, it gives your front-end devs similar environment to create the templates, rather than learning a new system/syntax (you can always write a bit of code to enable in-browser preview). Plus you get intellisense.

Parsing Your Templates
-------------------

First of all, create a model for your template

{% highlight c# %}
public class EmailModelBase
{
    public string Name { get; set; }
    public string UnsubscribeUrl { get; set; }
}
{% endhighlight %} 

You can then read the content of your template and use your model to parse the resulting HTML by calling **Razor.Parse()**

{% highlight c# %}
public static class EmailFactory
{
     public static string ParseTemplate<T>(T model, string pathToTemplates, EmailType emailType)
     {
         var templatePath = Path.Combine(pathToTemplates, string.Format("{0}.cshtml", emailType));
         var content = templatePath.ReadTemplateContent();
 
         return Razor.Parse(content, model);
     }
}
{% endhighlight %} 

Editing your HTML Templates becomes a doddle – just use the @Model syntax

{% highlight c# %}
... more HTML
<p align="left" class="article-title">Dear @Model.Name</p>
... more HTML
{% endhighlight %}  

If you need to, you can use loops or conditional statements

{% highlight c# %}
@foreach(var image in Model.Images)
{
   <td class="w130" width="130" valign="top">
       <table class="w130" width="130" cellpadding="0" cellspacing="0" border="0">
           <tbody><tr>
                      <td class="w130" width="130"><img src="@image" editable="true" label="Image" class="w130" width="130" border="0"></td>
                  </tr>
           </tbody></table>
   </td> 
}
{% endhighlight %} 

Building New Email Templates
-------------------

Building new html email templates can be very tedious. All the style needs to be inline and getting things working well cross device can be a pain. Campaign monitor has a simple email builder that you can base your emails on -  [Campaign Monitor Email Builder](https://templates.campaignmonitor.com/build/). If you want test cross-device, [Litmus is a place to go](http://litmus.com/). You can send a test email to litmus and it gives you a preview on huge variety of devices.

Testing Your Email Templates Locally
-------------------

Rather than setting up SMTP on your local machine, I suggest you use something like [SMTP 4 Dev](http://smtp4dev.codeplex.com/). It runs in the tray, listens to the port 25 and catches all your outgoing emails. It also lets you preview each email. This means your local system can send out emails without you worrying they will be delivered and you can preview them easily (providing you use System.Net.Mail and your Smtp client or web.config host is set to localhost).

Sample App
-------------------

The sample app illustrates couple of examples, gives you the ability to generate, preview and send Razor Email templates via SMTP client.  You can download the app from the link below.

<iframe height="120" src="https://skydrive.live.com/embed?cid=84E23A97F665C5F2&amp;resid=84E23A97F665C5F2%21240&amp;authkey=AAj4uKGWd7hLe08" frameborder="0" width="98" scrolling="no"></iframe>

Any questions, give me a shout [@mirajavora](http://twitter.com/mirajavora)