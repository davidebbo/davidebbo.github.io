---
layout: post
title:  "App_Start folder convention for NuGet and WebActivator"
comments: true
categories: [WebActivator,NuGet]
---




[Please see the [WebActivator wiki](https://bitbucket.org/davidebbo/webactivator/wiki/Home) for the latest docs]
When I first [blogged](http://blogs.msdn.com/b/davidebb/archive/2010/10/11/light-up-your-nupacks-with-startup-code-and-webactivator.aspx) about WebActivator, I showed in my example using a source file named AppStart_SparkMvc.cs.pp under the Content folder in the package, which means when you install it you end up with a file named AppStart_SparkWebMvc.cs at the root of your web project.


Now suppose you install a few more packages that use the same WebActivator pattern, and you would end up with something like that at the root of your project:

```
AppStart_SparkWebMvc.cs
AppStart_SQLCEEntityFramework.cs
AppStart_BarPackage.cs
AppStart_BlahPackage.cs
Global.asax
Global.asax.cs
Web.config
More files...

```

That starts getting really ugly, and most devs like to keep the root of their app free of clutter.

## We need a better convention!

The solution is simply to agree on a different convention where we put all this startup code into a folder. To match ASP.NET conventions, the obvious name to pick is App_Start. And once we do that, we no longer need to prefix the file names with AppStart, so we would have:

```
App_Start
SparkWebMvc.cs
SQLCEEntityFramework.cs
BarPackage.cs
BlahPackage.cs
Global.asax
Global.asax.cs
Web.config
More files...

```

Likewise, the full class names would change from WebApplication1.AppStart_SQLCEEntityFramework to WebApplication1.App_Start.SQLCEEntityFramework. Note that the namespace doesn't matter a whole lot since you won't call this code explicitly. But since existing convention is to have the namespace match the folder structure, we may as well do that here.

As of today, there are 17 packages that use WebActivator, so I'll need to try to convince all the authors to follow this. Fun time ahead! :)

But note that it's just a convention, with no code changes to enforce it. Nothing written here breaks any existing packages. It's just something where by agreeing on a better convention, we make NuGet yet a little bit better!

## An example: EFCodeFirst.SqlServerCompact

As an example, here is what I ended up with for the EFCodeFirst.SqlServerCompact package using this pattern.

The source file transform in the package is in Content\App_Start\SQLCEEntityFramework.cs.pp, and contains:

{% highlight c# %}
// namespaces, etc...

[assembly: WebActivator.PreApplicationStartMethod(
typeof($rootnamespace$.App_Start.SQLCEEntityFramework), "Start")]

namespace $rootnamespace$.App_Start {
public static class SQLCEEntityFramework {

//etc...
{% endhighlight %}
Note the use of $rootnamespace$ and of App_Start in the namespace.
