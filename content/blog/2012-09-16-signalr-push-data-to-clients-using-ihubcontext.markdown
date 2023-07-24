---
layout: post
title: "SignalR – Push Data To Clients Using IHubContext"
date: 2012-09-16 12:37:00 +0000
comments: true
summary: "The Clients dynamic property of the Hub gives you access to all clients connected to the hub within the hub class. However, what if you would like to push data to the clients from outside of the Hub class. One of the most common scenarios is when you want to push data to the clients from an admin system in your back-end."
categories: [C#, SignalR, Asp.Net, Visual Studio]
---

*This article is a second contribution to the series dedicated to SignalR. My previous article focused on introduction and basics of SignalR.*

Access to the Hubs from outside of the Hub class
-------------------

The Clients dynamic property of the Hub gives you access to all clients connected to the hub within the hub class. However, what if you would like to push data to the clients from outside of the Hub class. One of the most common scenarios is when you want to **push data to the clients from an admin** system in your back-end.

This is where the static **GlobalHost** SignalR class comes to rescue. It gives you access to the HubContext through the IConnectionManager interface.

{% highlight c# %}
var myHub = GlobalHost.ConnectionManager.GetHubContext<FeedHub>();
var result = myHub.Clients.receiveData(yourData);
{% endhighlight %}

The IHubContext, which is returned from the GetHubContext<T> exposes the **dynamic Clients and IGroupManager Groups**. This means that you can get access to the clients connected to the hubs from anywhere in your app. Pretty cool.

Extend our example to publish from controllers
-------------------

Lets assume, we want to add functionality to our [existing example](/signalr-introduction-to-signalr-quick-chat-app) to publish data from some sort of mangement/admin controller. All we need is a new hub (FeedHub), the management controller and a bit of Javascript to wire it all together.

Lets create the hub first. It is actually just a proxy between the JS client and the controller so it could be empty since all methods are called via the Clients dynamic variable.

{% highlight c# %}
public class FeedHub : Hub
{
 
}
{% endhighlight %} 

Then, we need a controller that will create and publish the items to the clients via the FeedHub.

{% highlight c# %}
public class ManageController : Controller
{
    private readonly IRepository _repository;
 
    public ManageController(IRepository repository)
    {
        _repository = repository;
    }
 
    public ActionResult Index()
    {
        var model = new ManageModel() {Items = _repository.FindAll<FeedItem>()};
        return View(model);
    }
 
    public ActionResult Add()
    {
        var model = new AddFeedItemModel();
        return View(model);
    }
 
    [HttpPost]
    public ActionResult Add(AddFeedItemModel model)
    {
        var update = new FeedItem() {Message = model.Content};
        _repository.Save(update);
 
        return RedirectToAction("Index");
    }
 
    public ActionResult Publish(long feedItemId)
    {
        var myHub = GlobalHost.ConnectionManager.GetHubContext<FeedHub>();
 
        var item = _repository.FindById<FeedItem>(feedItemId);
        item.IsPublished = true;
        _repository.Save(item);
 
        var result = myHub.Clients.receive(item);
        return RedirectToAction("Index");
    }
}
{% endhighlight %}

Finally, we need to wire-up the front-end with Javascript and template file to render the data coming from the server

{% highlight javascript %}
function Feed() {
    var feedHub = undefined;
 
    var init = function () {
        feedHub = $.connection.feedHub;
 
        feedHub.receive = function (item) {
            var selector = "ul.feed-list li[data-id=" + item.Id + "]";
            if (!($(selector).length > 0)) {
                $("ul.feed-list").prepend($(".feed-template").render(item));
                $("ul.feed-list li:gt(3)").remove();
            }
        };
    };
 
    init();
};
 

<script class="feed-template" type="text/x-jquery-tmpl">
    <li data-id="{{>Id}}">
        <div class="row-fluid">
            <div class="span8">
                <h3>{{>Message}}</h3>
            </div>
        </div>
    </li>
</script>
{% endhighlight %}
 

Simple as that – the static GlobalHost class gives you the full power to push data to clients via any existing hub. Any questions, ping me [@mirajavora](http://twitter.com/mirajavora)

You can download the entire sample using the link below. You should change the connection string in web.config and run /boot to generate the DB schema. Remember that the sample is intended as a showcase of what SignalR can do and deliberately breaks several patterns for simplicity.

<iframe height="120" src="https://skydrive.live.com/embed?cid=84E23A97F665C5F2&amp;resid=84E23A97F665C5F2%21236&amp;authkey=AOXwZT1sff9A0Cg" frameborder="0" width="98" scrolling="no"></iframe>

Related Articles
-------------------

[SignalR – Introduction To SignalR](/signalr-introduction-to-signalr-quick-chat-app/)<br/>
[SignalR – Publish Data Using IHubContext](/signalr-push-data-to-clients-using-ihubcontext/)<br/>
[SignalR – Publish Data Using Proxies](/signalr-publish-data-from-win-forms-using-hub-proxies/)<br/>
[SignalR – Dependency Injection](/signalr-dependency-injection/)<br/>