---
layout: post
title: "Content Negotiation with ASP.NET Web API"
date: 2013-02-02 18:25:00 +0000
comments: true
image: /images/posts/webapi/FiddleQR_thumb.png
summary: "An important part of Web API is resource content negotiation. The HTTP protocol RFC defines content negotiation as the process of selecting the best representation for a given response when there are multiple representations available. In practise the same resource can be represented in a variety of different ways – lets say a contact information resource can be shown in JSON representation, but also in XML or even as a PNG QR code containing the same content."
categories: [WebAPI, Asp.Net, MVC, C#]
---

*This is a second post in the series all about ASP.NET Web API. The first post looked at [getting started with the Web API](http://blog.mirajavora.com/getting-started-with-asp.net-web-api). You can find the [source code on Github](https://github.com/mirajavora/WebAPISample).*

<a href="/images/posts/webapi/FiddleQR.png"><img alt="Fiddler QR" src="/images/posts/webapi/FiddleQR_thumb.png" class="post-image-right" /></a>
An important part of Web API is resource content negotiation. The HTTP protocol RFC defines content negotiation as **the process of selecting the best representation for a given response when there are multiple representations available**. In practise the same resource can be represented in a variety of different ways – lets say a contact information resource can be shown in JSON representation, but also in XML or even as a PNG QR code containing the same content.

The content negotiation can be either server or client driven. In the first instance, the server decides what content to send down based on the various headers sent from the client. The latter requires an additional call to the server to get the list of available representations. Web API provides you with an easy way to send different representations of the same content.

Creating new MediaTypeFormatter to Deliver QR Code
-------------------

The Web API comes with [MediaTypeFormatter](http://msdn.microsoft.com/en-us/library/system.net.http.formatting.mediatypeformatter.aspx) base class. If you want to create your own formatter, simply inherit from MediaTypeFormatter. If we take the [existing scenario of contact management](/getting-started-with-asp.net-web-api/), lets say we want to create a QR code that contains the contact information of each contact stored in the database.

Our QrMediaFormatter can only be used for writing the type of Contact. Therefore we add logic to the CanWriteType and override the WriteToStreamAsync member.

{% highlight c# %}
public class QrMediaFormatter : MediaTypeFormatter
{
    private const string ApiEndpoint = "http://chart.apis.google.com/chart";
 
    public override bool CanReadType(Type type)
    {
        return false;
    }
 
    public override bool CanWriteType(Type type)
    {
        if(type == null)
        {
            throw new ArgumentNullException("type");
        }
 
        return type == typeof(Contact);
    }
 
    public override Task WriteToStreamAsync(Type type, object value, Stream stream, System.Net.Http.HttpContent content, System.Net.TransportContext transportContext)
    {
        return Task.Factory.StartNew(() => WriteQrStream(type, value, stream, content.Headers));
    }
 
    private void WriteQrStream(Type type, object value, Stream stream, HttpContentHeaders contentHeaders)
    {
        var contact = value as Contact;
        var values = new Dictionary<string, string>()
                         {
                             {"cht", "qr"},
                             {"chs", "300x300"},
                             {"chld", "H|0"},
                         };
        values.Add("chl", String.Format("{0}%0D%0A{1}", HttpUtility.UrlEncode(contact.Name), HttpUtility.UrlEncode(contact.Email)));
        var endPointUrl = BuildUrl(values);
 
        using(var client = new WebClient())
        {
            var data = client.DownloadData(endPointUrl);
            stream.Write(data, 0, data.Length);
        }
    }
 
    private string BuildUrl(IDictionary<string, string> values)
    {
        return string.Concat(ApiEndpoint, QueryString(values));
    }
 
    private string QueryString(IDictionary<string, string> queryStringItems)
    {
        var stringBuilder = new StringBuilder();
        var joinCharacter = "?";
        foreach (var key in queryStringItems.Keys)
        {
            stringBuilder.AppendFormat("{0}{1}={2}", joinCharacter, key, queryStringItems[key]);
            joinCharacter = "&";
        }
 
        return stringBuilder.ToString();
    }
}
{% endhighlight %} 

The WriteQrStream method simply takes the Contact, builds up a simple string of the contact name and email and sends the string to the Google QR chart generation API. The stream coming from the API is then directly written to the response stream.

Wire Up Your Custom Media Type Formatter
-------------------

The **SupportedMediaTypes** property contains a collection of **MediaTypeHeaderValues**. These define which media types the formatter can handle. In our case we are happy just to map the QR formatter to a single type –> image/png. However, it is possible to have the same media formatter handle a variety of MediaTypeHeaderValues. For example image/png and image/jpeg.

{% highlight c# %}
public QrMediaFormatter()
{
    SupportedMediaTypes.Add(new MediaTypeHeaderValue("image/png"));   
}
{% endhighlight %} 

Once we have the media type in the SupportedMediaTypes, the client can use the Content-Type request header to request the image representation of the resource. You can try it out in fiddler, accessing the contact resource with **"Content-Type: image/png"** header.

Map Extension to the Media Type Formatter
-------------------

The idea is that the **same resource should not change URI based on representation**. However, it is difficult when some clients are unable to make requests for specific content type. For example, even if you use your contact resource as a path to an image /api/contact/a85ad33c-b61f-4503-8d75-861f3701efe3, the browser client does not send a content-type request for image and the service will the return a default representation of the resource, which in our case isn’t an image.

The way around it is to map a particular extension to the Media Type Formatter. It means adding an extension and therefore changing the URI, but it means it can be used by clients without specifying the requested content-type. First, you need to make sure your routes support extensions.

{% highlight c# %}
config.Routes.MapHttpRoute(
    name: "IdWithExt",
    routeTemplate: "api/{controller}/{id}.{ext}");
{% endhighlight %} 

Then you need to add UriPathExtensionMapping to the MediaTypeMappings.

{% highlight c# %}
public QrMediaFormatter()
{
    MediaTypeMappings.Add(new UriPathExtensionMapping("png", "image/png"));
    SupportedMediaTypes.Add(new MediaTypeHeaderValue("image/png"));   
}
{% endhighlight %} 

Once you have both in place, you should be able to request the resource on /api/contact/a85ad33c-b61f-4503-8d75-861f3701efe3.png URI.

Change the Configuration to Add the QR Formatter
-------------------

Finally, it is important to wire up all custom media formatters on app startup.

{% highlight c# %}
GlobalConfiguration.Configuration.Formatters.Add(new QrMediaFormatter());
{% endhighlight %} 

Code Sample
-------------------

You can check out all the the above in the [**code sample on GitHub**](https://github.com/mirajavora/WebAPISample). If you have any questions **give me a shout [@mirajavora](http://twitter.com/mirajavora)**