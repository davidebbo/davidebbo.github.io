---
layout: post
title:  "New features in WebActivator 1.4"
comments: true
tags: [WebActivator,NuGet]
---


[Please see the [WebActivator wiki](https://bitbucket.org/davidebbo/webactivator/wiki/Home) for the latest docs]

Back in October, I [blogged](http://blogs.msdn.com/b/davidebb/archive/2010/10/11/light-up-your-nupacks-with-startup-code-and-webactivator.aspx) about the WebActivator NuGet package, which allows packages to bring in some source code that runs on startup in a Web Application. It's been a pretty popular package, as there are many scenarios where running startup logic is important. The alternative of forcing the user to modify their global.asax is just not compelling.

There have also been a few feature requests since the initial 1.0 release, and I will describe them here.

## Ability to run code after Application_Start

When you use a WebActivator PreApplicationStartMethod attribute, the method it points to runs before your global.asax's. Yep, that's fairly obvious from the name Pre ApplicationStart :)

But in some cases, that's just too early. Scott Hanselman ran into that when trying to register MVC areas, and I added this feature as a result (see [his post](http://www.hanselman.com/blog/UpdatingAndPublishingANuGetPackagePlusMakingNuGetPackagesSmarterAndAvoidingSourceEditsWithWebActivator.aspx)).

This feature works exactly the same as the PreApplicationStartMethod attribute, except using a different attribute named… drums rolling… **Post**ApplicationStartMethod! e.g.

{% highlight c# %}
[assembly: WebActivator.PostApplicationStartMethod(
typeof(TestLibrary.MyStartupCode), "CallMeAfterAppStart")]

{% endhighlight %}

So when does that run exactly? It runs at the time the very first HttpModule get initialized. Internally, it's using the dynamic module registration mechanism [I blogged about recently](http://blog.davidebbo.com/2011/02/register-your-http-modules-at-runtime.html).

## Ability to run code when the app shuts down

WebActivator can also help you execute cleanup logic when the app shuts down. This is done via yet another attribute that works much like the other two, e.g.

{% highlight c# %}
[assembly: WebActivator.ApplicationShutdownMethod(
typeof(TestLibrary.MyStartupCode), "CallMeWhenAppEnds")]

{% endhighlight %}

This code runs at the time Dispose is called on the last HttpModule in the app.

## Support for code in App_Code in Web Sites

In a Web Site (as opposed to a Web Application), you typically put your shared code in the App_Code folder. Now if you have code in there that uses the PostApplicationStartMethod attribute, it will get called when the app starts, giving Web Sites some WebActivator love.

Please note that you can only use PostApplicationStartMethod in App_Code, and not PreApplicationStartMethod. The reason is that when PreApplicationStartMethod fires, the App_Code folder has not even been compiled!

## Support for invoking the start methods outside of ASP.NET

This change came courtesy of [Jakub Konecki](http://stackoverflow.com/users/449906/jakub-konecki), who needed it for unit testing purpose. This comes as a set of static methods that you can use to invoke the startup methods:

{% highlight c# %}
// Run all the WebActivator PreStart methods
WebActivator.ActivationManager.RunPreStartMethods();

// Run all the WebActivator PostStart methods
WebActivator.ActivationManager.RunPostStartMethods();

// Run all the WebActivator start methods
WebActivator.ActivationManager.Run();

// Run all the WebActivator shutdown methods
WebActivator.ActivationManager.RunShutdownMethods();

{% endhighlight %}
You can find the WebActivator sources [on bitbucket](https://bitbucket.org/davidebbo/webactivator).
