---
layout: post
title:  "Using Dynamic Data with EF Code First and NuGet"
comments: true
categories: [NuGet,DynamicData,EntityFramework]
---


Note: this post is a bit outdated. Checkout [this other post](http://blogs.msdn.com/b/webdev/archive/2012/08/15/using-dynamic-data-with-entity-framework-dbcontext.aspx) for more up to date information on this topic.

Dynamic Data works out of the box with Entity Framework, but it takes a small trick to get it working with the latest EF Code First bits (known as CTP5).

Here is quick walk through of what you need to do.

As a first step, create a new ASP.NET Dynamic Data Entities Web Application. Then, let's use [NuGet](http://nuget.org/) to add EF Code First to your project (I never miss a chance to pitch my new product!). We'll use it with SQL Compact, and also bring in a sample to get started.

Right click on References and choose 'Add Library Package Reference' to bring in the NuGet dialog. Go to the Online tab and type 'efc' (for EFCodeFirst) in the search box. Then install the EFCodeFirst.SqlServerCompact and EFCodeFirst.Sample packages:

![image](http://lh6.ggpht.com/_jySMpScpTXc/TT3cno7aujI/AAAAAAAAAUE/nFAKmRzp-Bw/image_thumb%5B7%5D.png?imgmax=800)

Now we need to register our context with Dynamic Data, which is the part that requires special handling. The reason it doesn't work the 'usual' way is that when using Code First, your context extends DbContext instead of ObjectContext, and Dynamic Data doesn't know about DbContext (as it didn't exist at the time).

I will show you two different approaches. The first is simpler but doesn't work quite as well. The second works better but requires using a new library.

## Approach #1: dig the ObjectContext out of the DbContext

The workaround is quite simple. In your RegisterRoutes method in global.asax, just add the following code (you'll need to import System.Data.Entity.Infrastructure and the namespace where your context lives):

```
public static void RegisterRoutes(RouteCollection routes) {
DefaultModel.RegisterContext(() => {
    return ((IObjectContextAdapter)new BlogContext()).ObjectContext;
}, new ContextConfiguration() { ScaffoldAllTables = true });


```

So what this is really doing differently is provide a Lambda that can dig the ObjectContext out of your DbContext, instead of just passing the type to the context directly.

And that's it, your app is ready to run!

![image](http://lh3.ggpht.com/_jySMpScpTXc/TT3f-mzXXNI/AAAAAAAAAUM/gA2W27FTzD4/image_thumb%5B9%5D.png?imgmax=800)

One small glitch you'll notice is that you get this EdmMetadatas entry in the list. This is a table that EF creates in the database to keep track of schema versions, but since we told Dynamic Data to Scaffold All Tables, it shows up. You can get rid of it by turning off ScaffoldAllTables, and adding a [ScaffoldTable(true)] attribute to the entity classes that you do want to see in there.

Another issue is that this approach doesn't work when you need to register multiple models, due to the way the default provider uses the ObjectContext type as a key. Since we don't actually extend ObjectContext, all contexts end up claiming the same key.

## Approach #2: use the DynamicData.EFCodeFirstProvider library

This approach is simple to use, but just requires getting a library with a custom provider. If you don't already have NuGet, get it from [here](http://nuget.org/).

Then install the DynamicData.EFCodeFirstProvider package in your project:

```
PM> Install-Package DynamicData.EFCodeFirstProvider
'EFCodeFirst 0.8' already installed.
Successfully installed 'DynamicData.EFCodeFirstProvider 0.1.0.0'.
WebApplicationDDEFCodeFirst already has a reference to 'EFCodeFirst 0.8'.
Successfully added 'DynamicData.EFCodeFirstProvider 0.1.0.0' to WebApplicationDDEFCodeFirst.


```

After that, this is what you would write to register the context in your global.asax:

{% highlight c# %}
DefaultModel.RegisterContext(
   new EFCodeFirstDataModelProvider(() => new BlogContext()),
   new ContextConfiguration() { ScaffoldAllTables = true });

{% endhighlight %}
And that's it! This approach allows registering multiple contexts, and also fixes the issue mentioned above where EdmMetadatas shows up in the table list.
  
