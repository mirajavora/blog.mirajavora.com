---
layout: post
title: "SignalR-Dependency Injection"
date: 2012-10-03 21:10:00 +0000
comments: true
summary: "A larger web projects would typically consist not only of front end web project, but would include additional class libraries and offload some of the heavy processing work to service or console apps. The common problem is then how do you update the front-end and signal the site that some work has been completed."
tags: [C#, SignalR, Asp.Net, Visual Studio]
archive: 2012
---

*This is a fourth contribution in a series dedicated to SignalR. The previous post focused on publishing data from .net services using hub proxies.*

Setting up dependency injection with SignalR is pretty straightforward. SignalR uses DefaultDependencyResolver to resolve its own services (IConnectionManager, IConfigurationManager, IJsonSerializer, ITransportManager and many more) and hub extensions (IHubManager, IJavaScriptProxyGenerator and many more). Using GlobalHost, you can set your own dependency resolver.

Extending the DefaultDependencyResolver
-------------------

When it comes to writing your own dependency resolver for SignalR, you have two options: 
- You can implement IDependencyResolver  and write the entire logic yourself (for the brave with lots of time)
- You can extend the DefaultDependencyResolver and override members with your own logic (the suggested route)

The DefaultDependencyResolver Members

{{< highlight csharp "linenos=inline" >}}
public virtual object GetService(Type serviceType);
public virtual IEnumerable<object> GetServices(Type serviceType);
public virtual void Register(Type serviceType, Func<object> activator);
public virtual void Register(Type serviceType, IEnumerable<Func<object>> activators);
{{< / highlight >}} 

All you need to do when extending the DefaultDependencyResolver is to override the GetService and GetServices methods. In the example below, I’m using Castle.Windsor container, but similar pattern can be used with any other DI containers. You can find few existing implementations below.

{{< highlight csharp "linenos=inline" >}}
public class SignalrDependencyResolver : DefaultDependencyResolver
{
    private readonly IKernel _kernel;
 
    public SignalrDependencyResolver(IKernel kernel)
    {
        _kernel = kernel;
    }
 
    public override object GetService(System.Type serviceType)
    {
        //check if component exists in your container, if not use base to resolve
        return _kernel.HasComponent(serviceType) ? _kernel.Resolve(serviceType) : base.GetService(serviceType);
    }
 
    public override IEnumerable<object> GetServices(Type serviceType)
    {
        var objects = _kernel.HasComponent(serviceType) ? _kernel.ResolveAll(serviceType).Cast<object>() : new object[] { };
        return objects.Concat(base.GetServices(serviceType));
    }
}
{{< / highlight >}} 

In order to wire up your custom DependencyResolver, you need to set the DependencyResolver via GlobalHost.

{{< highlight csharp "linenos=inline" >}}
protected void Application_Start()
{
    var container = CreateContainer();    
    //container init code ...
 
    //create dependency resolver
    var signalrDependency = new SignalrDependencyResolver(container.Kernel);
    GlobalHost.DependencyResolver = signalrDependency;
    RouteTable.Routes.MapHubs(signalrDependency);
 
    //further startup code
    //...
}
{{< / highlight >}} 

With the setup above, you can register all your hubs in your container.

{{< highlight csharp "linenos=inline" >}}
public class HubsInstaller : IWindsorInstaller
{
    public void Install(IWindsorContainer container, IConfigurationStore store)
    {
        container.Register(
            AllTypes.FromAssembly(typeof(MvcApplication).Assembly)
                .BasedOn<Hub>().Configure(
                    x => x.Named(x.Implementation.FullName.ToLower()).LifeStyle.Transient));
    }
}
{{< / highlight >}} 
 

Your container will then auto-resolve any services that are already registered.

{{< highlight csharp "linenos=inline" >}}
public class ChatHub : Hub
{
    private readonly IRepository _repository;
 
    public ChatHub(IRepository repository)
    {
        _repository = repository;
    }
 
    //methods ...
}
{{< / highlight >}} 
 

Under The Hood Of DefaultDependecyResolver
-------------------

You can check out the default dependency resolver at GitHub.  You can find all the services that are registered – the resolver keeps then in a private readonly dictionary Dictionary<Type, IList<Func<object>>>. If you fancy replacing an internal part of SignalR with your own implementation, you can locate the correct interface and register your own implementation in your container. The custom dependency resolver will then resolve your implementation.

Existing SignalR Dependency Injection Projects
-------------------

[SignalR Unity](https://github.com/bradygaster/SignalR.Unity/)<br/>
[SignalR Ninject](https://github.com/SignalR/SignalR.Ninject/)<br/>
[SignalR Autofac](https://github.com/pszmyd/SignalR.Autofac)<br/>

Related Articles
-------------------

[SignalR – Introduction To SignalR](/signalr-introduction-to-signalr-quick-chat-app/)<br/>
[SignalR – Publish Data Using IHubContext](/signalr-push-data-to-clients-using-ihubcontext/)<br/>
[SignalR – Publish Data Using Proxies](/signalr-publish-data-from-win-forms-using-hub-proxies/)<br/>
[SignalR – Dependency Injection](/signalr-dependency-injection/)<br/>

Any questions or comments, give me a shout [@mirajavora](http://twitter.com/mirajavora)