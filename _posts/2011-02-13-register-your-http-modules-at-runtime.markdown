---
layout: post
title:  "Register your HTTP modules at runtime without config"
comments: true
categories: ASP.NET
---


In ASP.NET 4, we added the concept of a PreApplicationStart method that an assembly can use to execute code early on in the appdomain without any configuration. Phil Haack covered it a while back in [this post](http://haacked.com/archive/2010/05/16/three-hidden-extensibility-gems-in-asp-net-4.aspx). It's pretty simple to use. You just define a class that looks like:

{% highlight c# %}
public class PreApplicationStartCode {
public static void Start() {
    // Your startup code here
}
}

{% endhighlight %}

And then you add an assembly level attribute pointing to it:

{% highlight c# %}
[assembly: PreApplicationStartMethod(typeof(PreApplicationStartCode), "Start")]

{% endhighlight %}

With the release of MVC3 and ASP.NET Web Pages, we added another little gem: a RegisterModule() API that lets you dynamically register an IHttpModule without touching config. Sadly, the method is hidden so deep that it is hard to find by accident (it'll get cleaned up in the next framework version).

By combining the two techniques, you have everything you need to register a module dynamically, e.g.

{% highlight c# %}
public class PreApplicationStartCode {
public static void Start() {
    // Register our module
    Microsoft.Web.Infrastructure.DynamicModuleHelper.DynamicModuleUtility.RegisterModule(typeof(MyModule));
}
}
{% endhighlight %}

I warned you it was well hidden! :)

The module type that you pass in to that is just a standard IHttpModule, e.g. here is a basic module that writes to the response on every request:

{% highlight c# %}
class MyModule : IHttpModule {
public void Init(HttpApplication context) {
    context.BeginRequest += (sender, e) => {
        var response = ((HttpApplication)sender).Response;
        response.Write("MyModule.BeginRequest");
    };
}

public void Dispose() { }
}

{% endhighlight %}

The beauty of this is that it allows you to create fully encapsulated assemblies that you can just drop into a web app's bin folder and have them light up without having to add any ugly registration to the app.

And yes, all this works fine in partial trust!

You can download a minimal sample from [here](https://docs.google.com/uc?id=0B9LFjrvVZR24ZGQ4ZWY3YjYtN2Y3NC00ODcyLTlmMDktNWUxNWM0ZmM2ZjAw&amp;export=download&amp;hl=en).

