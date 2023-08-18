---
layout: post
title: "Auto Wire-Up Your Model Binders ASP.NET MVC"
date: 2012-07-08 12:25:00 +0000
comments: true
summary: "Model binding in MVC attempts to map values from IValueProviders to your specified model.  The value providers (FormValueProvider, QueryStringValueProvider, HttpFileCollectionValueProvider, RouteDataValueProvider, JsonValueProviderFactory … ) abstract the actual value retrieval and binders then handle the value mapping onto the model. You can create custom model binders to abstract and centralise complex mapping logic that would otherwise end up in your controllers or services. When dealing with a larger number of binders, it is good to refactor common logic and enable wiring up of the new binders easily. Instead of having to register each binder one-by-one in global or prefix the type in the actions, we can create our own model binder broker to replace the default MVC DefaultModelBinder."
tags: [C#, MVC, ASP.NET, Visual Studio]
archive: 2012
---

Model binding in MVC attempts to map values from IValueProviders to your specified model.  The value providers (FormValueProvider, QueryStringValueProvider, HttpFileCollectionValueProvider, RouteDataValueProvider, JsonValueProviderFactory … ) abstract the actual value retrieval and binders then handle the value mapping onto the model.

You can create custom model binders to abstract and centralise complex mapping logic that would otherwise end up in your controllers or services. When dealing with a larger number of binders, it is good to refactor common logic and enable wiring up of the new binders easily. Instead of having to register each binder one-by-one in global or prefix the type in the actions, we can create our own model binder broker to replace the default MVC DefaultModelBinder.
<!--more-->

A Common Scenario
-------------------

Retrieving a persisted domain object from the database that is then passed to the ViewModel. Lets assume we have a database of books and we want to get a single book by it’s ID.

{{< highlight csharp "linenos=inline" >}}
public ActionResult Book(Guid? id)
{
    if (!id.HasValue) 
        return HandleInvalidBook();
 
    var book = _repository.FindById<Book>(id);
    if (book == null)
        return HandleInvalidBook();
 
    var model = new BookModel() { Book = book };
    return View(model);
}
{{< / highlight >}}

Binder Code
-------------------

Rather than doing all the lookup work in the controller, you can create a BookBinder that will call the repo and retrieve the Book object by given ID. The BinderBase expects the type of the domain object and the type of the ID. It auto-converts the value from IValueProviders into the expected ID type and calls BindModelWithId.

{{< highlight csharp "linenos=inline" >}}
public class BookBinder : BinderBase<Book, Guid>
{
    private readonly IRepository _repository;
 
    public BookBinder(IRepository repository)
    {
        _repository = repository;
    }
 
    protected override object BindModelWithId(ControllerContext controllerContext, ModelBindingContext bindingContext, Guid id)
    {
        return _repository.FindById<Book>(id);
    }
}
 
public abstract class BinderBase<T, TId> : IFilteredBinder
{
    protected virtual string ValueIdentifier
    {
        get { return "id"; }
    }
 
    public object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
    {
        var itemId = GetObjectId(controllerContext, bindingContext);
        return BindModelWithId(controllerContext, bindingContext, itemId);
    }
 
    protected virtual TId GetObjectId(ControllerContext controllerContext, ModelBindingContext bindingContext)
    {
        var value = bindingContext.ValueProvider.GetValue(ValueIdentifier);
        return value == null ? default(TId) : Convert<TId>(value.AttemptedValue);
    }
 
    protected abstract object BindModelWithId(ControllerContext controllerContext, ModelBindingContext bindingContext, TId id);
 
    public Type ShouldBind
    {
        get { return typeof (T); }
    }
 
    private static TId Convert<TId>(string input)
    {
        try
        {
            var converter = TypeDescriptor.GetConverter(typeof(TId));
            if (converter != null)
            {
                return (TId) converter.ConvertFromString(input);
            }
            return default(TId);
        }
        catch
        {
            return default(TId);
        }
    }
}
 
public interface IFilteredBinder : IModelBinder
{
    Type ShouldBind { get; }
}
{{< / highlight >}}


Using Model Binders To Retrieve The Book Object
-------------------

You can then use your binder on your Controller Action

{{< highlight csharp "linenos=inline" >}}
public ActionResult Book(Book book)
{
    if (book == null)
        return HandleInvalidBook();
 
    var model = new BookModel() {Book = book};
    return View(model);
}
{{< / highlight >}}


A Smart Way To Wire-Up Your Model Binders
-------------------

Registering binders one-by-one in the Global.asax or prefixing them in the Action gets tedious and it’s easy to make a mistake. A smarter way is to implement a default model binder and decide which binder to use based on ShouldBind property of each IFilteredBinder.

{{< highlight csharp "linenos=inline" >}}
public class ModelBinderBroker : DefaultModelBinder
{
    private readonly IEnumerable<IFilteredBinder> _binders;
 
    public ModelBinderBroker(IEnumerable<IFilteredBinder> binders)
    {
        _binders = binders;
    }
 
    public override object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
    {
        if(_binders == null || _binders.Count() == 0)
            return base.BindModel(controllerContext, bindingContext);
 
        var binder = _binders.FirstOrDefault(x => x.ShouldBind == bindingContext.ModelType);
        return binder == null ? base.BindModel(controllerContext, bindingContext) : binder.BindModel(controllerContext, bindingContext);
    }
}
{{< / highlight >}}


Register Your Binder Broker In Global.asax
-------------------

You can then set your custom ModelBinderBroker as the DefaultBinder.

{{< highlight csharp "linenos=inline" >}}
protected void Application_Start()
{
    var container = new Castle.Windsor.WindsorContainer();
    container.Kernel.Resolver.AddSubResolver(new CollectionResolver(container.Kernel, true));<br>
    //binders
    container.Register(
        AllTypes.FromAssembly(typeof(IFilteredBinder).Assembly)
        .BasedOn<IFilteredBinder>()
    );
    container.Register(Component.For<ModelBinderBroker>().ImplementedBy<ModelBinderBroker>());
    container.Register(Component.For<IRepository>().ImplementedBy<FakeRepository>());
 
    RegisterBinders(container);
}
 
private void RegisterBinders(IWindsorContainer container)
{
    ModelBinders.Binders.DefaultBinder = container.Resolve<ModelBinderBroker>();
}
{{< / highlight >}}

Download code
-------------------

<iframe height="120" src="https://skydrive.live.com/embed?cid=84E23A97F665C5F2&amp;resid=84E23A97F665C5F2%21140&amp;authkey=AA9mcKzV4kTZFpQ" frameborder="0" width="98" scrolling="no"></iframe>

Related Posts
-------------------

Phil Haack – What is the difference between a value provider and model binder
[http://haacked.com/archive/2011/06/30/whatrsquos-the-difference-between-a-value-provider-and-model-binder.aspx)](http://haacked.com/archive/2011/06/30/whatrsquos-the-difference-between-a-value-provider-and-model-binder.aspx)

Mehdi Golchin – Deep Dive Into Value Providers
[http://mgolchin.net/posts/19/dive-deep-into-mvc-ivalueprovider](http://mgolchin.net/posts/19/dive-deep-into-mvc-ivalueprovider)