---
layout: post
title: "SignalR – Introduction to SignalR – Quick Chat App"
date: 2012-09-05 16:04:00 +0000
comments: true
image: /images/posts/signalr/NuGet_thumb.png"
summary: "SignalR is an open source async signalling library. It was made by David Fowler and Damian Edwards. In a nutshell, it allows you to pass data between client and server in realtime. It’s not only for web, it has JS, .Net, WP7 and Silverlight clients and supports self-hosting so you can run the SignalR server in win service or web context. It will run on .Net 4.0 or 4.5 and to get websockets running, you will need IIS8 or IIS8 Express. That said, it will run on older versions of IIS and will switch to different transport modes."
categories: [C#, SignalR, Asp.Net, Visual Studio]
---

*This article is an introduction to a series I will be publishing over the next few weeks. All based on SignalR and common scenarios when utilising this awesome library in web development.*

What is SignalR
-------------------

SignalR is an [open source](https://github.com/SignalR/SignalR) async signalling library. It was made by [David Fowler](http://twitter.com/davidfowl) and [Damian Edwards](http://twitter.com/DamianEdwards).

In a nutshell, it allows you to pass data between client and server in realtime. It’s **not only for web**, it has JS, .Net, WP7 and Silverlight clients and supports self-hosting so you can run the SignalR server in win service or web context. It will run on .Net 4.0 or 4.5 and to get websockets running, you will need IIS8 or IIS8 Express. That said, it will run on older versions of IIS and will switch to different transport modes.

Hubs and Persistent Connections
-------------------

SignalR talks about two concepts – **persistent connections** and **hubs**. Persistent Connection is a lower level API which is exposed over http. Hubs expose public methods to the clients and raise callbacks on the clients. In most web-based scenarios, you will be utilising hubs, which follow the publish-subscribe pattern.

Transport Modes Available
-------------------

When a connection between a web client and a server is made, SignalR will determine a suitable transport type based on your client capabilities. It will gracefully degrade so older browsers might get long-polling instead of the fancy websockets. The transport mode can have a significant impact on the performance of the app.

- **WebSockets** (bidirectional stream)
- **Server Sent Events** (push notifications from server to browser using DOM events)
- **Forever Frame** (uses HTTP 1.1 chunked encoding to establish a single long-lived HTTP connection in a hidden iframe)
- **Long polling** (hit the server hit the server hit the server hit the server hit server and hope something comes back with data)

Creating your first hub
-------------------

First, you will need to include SignalR in your project. The quickest way is to use NuGet. You will notice, there are various packages available. In this example, we are using both client and server parts of SignalR –> therefore you should choose the very first “SignalR” option. This will bring down all the dependencies.

![Nuget Feed Signalr](/images/posts/signalr/NuGet_thumb.png)

Creating a new hub then becomes very easy. Simply create a class and inherit from an abstract Hub class.

{% highlight c# %}
public class ChatHub : Hub
{
    //your code here ...
}
{% endhighlight %}

Connecting to the Hub from a web client

In order to connect to the hub, you will need to add couple of Javascript references. Please note that SignalR uses JQuery. Therefore if it’s not included already, you should include it before initialising signalR.

{% highlight html %}
....
    <script type="text/javascript" src="~/Scripts/jquery.signalR-0.5.3.js"></script>
    <script type="text/javascript" src="~/signalr/hubs"></script>
....
{% endhighlight %}
 

The ~/signalr/hubs endpoint exposes all the available hubs in the solution and lets you access their public methods. You will first need to initiate connection.

{% highlight javascript %}
$.connection.hub.start()
    .done(function() {
        console.log("Connected!");
    })
    .fail(function() { 
        console.log("Could not Connect!"); 
    });
{% endhighlight %}

Distributing your messages to the clients
-------------------

Once you have made a connection to the hub, your client can then call the public methods on the hub and subscribe to the callbacks received from the hub.

{% highlight c# %}
public class ChatHub : Hub
{
    private readonly IRepository _repository;
 
    public ChatHub(IRepository repository)
    {
        _repository = repository;
    }
 
    public void Send(string message)
    {
        var msg = new ChatMessage() {Message = message};
        _repository.Save(msg);
 
        Clients.receiveChat(msg);
    }
}
{% endhighlight %}

When you want to distribute messages to your clients, you can do so using the **Clients dynamic object**. Any method that you call on Clients will raise a callback on the client. Furthermore, you can get access to the current client call id using **Context.ConnectionId** or Groups dynamic object which looks after groups management. To publish on a specific connection you can you Clients[“gropuName”].method(params) or Clients[Context.ConnectionId].method(params).

Finally, you should register your client callbacks in JS and wire-up the buttons, fields and your preferred rendering template

{% highlight javascript %}
var chatHub = undefined;

var init = function () {
    $(".chat-submit").on("click", sendMessage);

    chatHub = $.connection.chatHub;

    chatHub.receiveChat = function (value) {
        console.log('Server called addMessage(' + value + ')');
        $("ul.chat-list").prepend($(".chat-template").render(value));
        $("ul.chat-list li:gt(2)").remove();
    };
};

var sendMessage = function() {
    chatHub.send($(".chat-message").val());
};
{% endhighlight %}
 

And that’s it, simple as that. You can check out the code sample below (it is a part of a bigger project and there is much more to come). Any questions, give me a shout @mirajavora

Related Links
-------------------

[SignalR on GitHub](https://github.com/SignalR/SignalR)

Download the code
-------------------

<iframe height="120" src="https://skydrive.live.com/embed?cid=84E23A97F665C5F2&amp;resid=84E23A97F665C5F2%21233&amp;authkey=AG12GlBD-u_3NBI" frameborder="0" width="98" scrolling="no"></iframe>

The sample app is based on MVC4 with Castle Windsor DI and FluentNhibernate. You can edit the connection string in web.config and run /boot to get the schema created.

Related Articles
-------------------

[SignalR – Introduction To SignalR](/signalr-introduction-to-signalr-quick-chat-app/)<br/>
[SignalR – Publish Data Using IHubContext](/signalr-push-data-to-clients-using-ihubcontext/)<br/>
[SignalR – Publish Data Using Proxies](/signalr-publish-data-from-win-forms-using-hub-proxies/)<br/>
[SignalR – Dependency Injection](/signalr-dependency-injection/)<br/>