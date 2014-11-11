---
layout: post
title: "Elegant App Settings using Castle Dictionary Adapter Factory"
date: 2011-11-16 11:53:00 +0000
comments: true
categories: [C#, Castle Windsor, ASP.NET]
---

Traditionally, settings in ASP.Net apps are stored AppSettings area of the app as a key-value store. More complex apps would create specific config sections. The app would then have a static settings wrapper that would read the content from the web.config.
<!--more-->


I don’t know how many times I’ve seen this or variations of this.

{% highlight c# %}
public static class Settings
{
    public static string SomeSetting
    {
        get { return ConfigurationManager.AppSettings["SomeSetting"] ?? string.Empty; }
    }
}
{% endhighlight %}


Reasons to avoid static settings class
-------------------

### Testability

By using static setting classes with app/web.config, you are restricting yourself (unless you do some further work) to a single key-value per project. Now what if you want to test the behaviour of your system with different settings values?

### Dependency injection

With static settings class, you stick with a single specific implementation rather than an interface that can later injected with any implementation according to your needs.

Reasons to user static settings class
-------------------

### Access in the views

One major advantage of static settings classes is the ability to access your settings easily in the views. Extra code scaffolding would be required if dependency injection is used.

The best of both worlds
-------------------

The way to get best of both worlds is to have an abstract settings interface, that is then passed on to the static settings class. Even better, using Castle Windsor IoC, you can use DictionaryAdapterFactory to load and parse your settings via factory.

First declare the IApplicationSettings interface

{% highlight c# %}
public interface IApplicationSettings
{
    string TestUrl { get; set; } 
}
{% endhighlight %}

Create a static class to mirror the settings. Or include the settings that you only need in your views / static context

{% highlight c# %}
public static class Settings
{
    private static IApplicationSettings _settings;
 
    public static void InitSettings(IApplicationSettings settings)
    {
        _settings = settings;
    }
 
    public static string TestUrl
    {
        get { return _settings.TestUrl; }
    }
}
{% endhighlight %}

Create the container and register the factory via DictionaryAdapterFactory (in your bootstrap or global.asax etc)

{% highlight c# %}
...
 
WindsorContainer container = new WindsorContainer();
container.AddFacility<Castle.Facilities.FactorySupport.FactorySupportFacility>();
 
container.Register(
    Component.For<IApplicationSettings>().UsingFactoryMethod(
        () => new DictionaryAdapterFactory()
             .GetAdapter<IApplicationSettings>(ConfigurationManager.AppSettings)));
...
{% endhighlight %}

Remember to instantiate your static settings class with the IApplicationSettings (after you registered the factory)

{% highlight c# %}
Settings.InitSettings(container.Resolve<IApplicationSettings>());
{% endhighlight %}

Have fun!