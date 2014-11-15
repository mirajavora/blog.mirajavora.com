---
layout: post
title: "SignalR - Publish Data From Win Forms Using Hub Proxies"
date: 2012-09-25 22:31:00 +0000
comments: true
summary: "A larger web projects would typically consist not only of front end web project, but would include additional class libraries and offload some of the heavy processing work to service or console apps. The common problem is then how do you update the front-end and signal the site that some work has been completed."
categories: [C#, SignalR, Asp.Net, Visual Studio, MVC]
---

*This article is a third in a series dedicated to SignalR. My previous article looked at pushing data using IHubContext.*

A larger web projects would typically consist not only of front end web project, but would include additional class libraries and offload some of the heavy processing work to service or console apps. The common problem is then **how do you update the front-end and signal the site that some work has been completed**.

A crude way around this is to store a flag in your persistence and continuously poll the data whether the job has finished. This is grossly inefficient. SignalR offers a straightforward solution to exactly that problem. Using Hub proxies, you are able to push data all the way to the connected clients on the front-end.

Image Hub
-------------------

In my example, I will use Win Forms app to poll Instagram for latest images based on a specific tag and push the latest images to the web clients using a hub proxy.

First of all, I need a Hub in the web project to which my clients will connect to.

{% highlight c# %}
public class ImagesHub : Hub
{
    public void PublishPhoto(string url)
    {
        Clients.receive(url);
    }
}
{% endhighlight %}  

The hub will receive the url of the latest image and pass it to the clients using the **Clients dynamic property**. We can add a bit of JavaScript to let the client connect to the hub and listen to the receive callback. Once the data is received on the client, I use jsrender to take the value and render the image in the div container.

{% highlight javascript %}
var imagesHub = undefined;

var init = function() {
    imagesHub = $.connection.imagesHub;

    imagesHub.receive = function (value) {
        $(".images").html($(".image-template").render({ Image: value }));
    };
};

init();
{% endhighlight %} 
 

Creating a Connection and a Proxy
-------------------

On the win forms side, we can then establish a hub connection. The HubConnection class takes url as its parameter. This is the url of your web project that contains the hubs. You can then create a proxy for the hub using the hub name and start the connection afterwards.

{% highlight c# %}
private HubConnection _connection;
private IHubProxy _imageHub;
 
private void Init()
{
    //...your init code  
 
    //create connection
    _connection = new HubConnection(EndPointUrl.Text);
    //create proxy to the ImagesHub
    _imageHub = _connection.CreateProxy("ImagesHub");
    //start the connection
    _connection.Start();
 }
{% endhighlight %}  

Invoke a method on the Hub using a Proxy
-------------------

Once you establish the connection, you will be able to use the hub proxies to invoke the PublishPhoto method on the ImagesHub. Calling Invoke method will execute the IHub method asynchronously and return the Task.

{% highlight c# %}
var image = GetImageToPublish(...);
 
_imageHub.Invoke("PublishPhoto", image).ContinueWith(task =>
{
    if (task.IsFaulted)
    {
         //your failure logic
    }
    else
    {
         //your success logic
    }
});
{% endhighlight %} 

Download the Sample
-------------------

The code is [available at GitHub](https://github.com/mirajavora/signalr-f1live), if you have any questions or comments give me a shout [@mirajavora](http://twitter.com/mirajavora)

Related Articles
-------------------

[SignalR – Introduction To SignalR](/signalr-introduction-to-signalr-quick-chat-app/)<br/>
[SignalR – Publish Data Using IHubContext](/signalr-push-data-to-clients-using-ihubcontext/)<br/>
[SignalR – Publish Data Using Proxies](/signalr-publish-data-from-win-forms-using-hub-proxies/)<br/>
[SignalR – Dependency Injection](/signalr-dependency-injection/)<br/>