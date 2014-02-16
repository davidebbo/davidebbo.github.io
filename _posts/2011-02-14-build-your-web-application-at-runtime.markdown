---
layout: post
title:  "Build your Web Application at runtime"
comments: true
tags: [ASP.NET]
---


**Disclaimer**: let me start by saying that the technique described in this blog is experimental, and is meant as a first step to see where this might take us. This is not in any way an officially supported technique!

If you are an ASP.NET user, you are likely aware that there are two different types of apps that you can create: Web Sites and Web Applications. Here is a quick summary of how they differ:

## Web Sites

In web sites, all compilation is done at runtime rather than design time. They don't use any VS project systems, and msbuild is never involved.

**Advantages**: very dynamic. You can just FTP files to the server, and everything just works. In that sense, it's similar to ASP Classic and PHP.

**Disadvantages**: lack of fine control over the build process; hard to unit test; often slower in VS; not available for MVC.

## Web Applications

In Web Applications, all the source code is built by VS in the designer using a standard .csproj file and msbuild. Pages and views (.aspx, .cshtml, …) are still built dynamically at runtime, so it's sort of a mixed mode model.

**Advantages**: full power of msbuild, easy to unit test code, fast build in VS.

**Disadvantages**: once you xcopy your built app to the server, you can't modify the code by just changing files (though you can still do this for pages &amp; views).

## What if we could get the best of both worlds?

I was recently chatting with my coworker [Louis DeJardin](http://twitter.com/#!/loudej) about compilation models, and he put out the idea that we might get something interesting if we were to run msbuild on the server, which is where this came from.

In a sense, it's sort of an 'obvious' thing to try if you look at the Pros can Cons of Web Sites and Web Applications. We want the full power of msbuild, but we also want the more dynamic nature of Web Sites, so the only logical thing to do is to run msbuild dynamically on the server!

## Try it now using NuGet!

Before I give you more details, let me show you how you can try this in no time via [NuGet](http://nuget.org/):
- Create a new MVC app
- Install my 'WebAppBuilder' NuGet package
- Run the app
- Change the message in Controllers\HomeController.cs, and **don't rebuild**
- Refresh the page in the browser (and then again per the message you'll get)
- Now try to make a change with a compile error and refresh again

## How does it all work?

There really isn't much code at all to make this work. First, it uses the technique I described in my previous post to [dynamically register a module](http://blog.davidebbo.com/2011/02/register-your-http-modules-at-runtime.html). This is what allows it to kick in without any registration.

Whenever the appdomain starts, the module looks for the csproj file and builds it. Doing this is quite simple since msbuild is well exposed to managed code (take a look at [Microsoft.Build.Execution.BuildManager](http://msdn.microsoft.com/en-us/library/microsoft.build.execution.buildmanager.aspx)). Note that it always does that on startup, with the assumption that the incremental build will be super fast if there is nothing to build.

Then if something actually got built, it sends back a simple page telling the user to refresh. This is a bit ugly as it effectively takes two refreshes to get the result, but it's necessary since we can't use the freshly built assembly in the same domain used to build it (since creating it causes a domain unload).

The other thing it does is listen to file change notification so it can unload the domain if any source files change. Then on the next request things get built as above.

There may be smarter ways of doing this, but this works pretty well as a proof of concept.

You can see [find code on bitbucket](https://bitbucket.org/davidebbo/webappbuilder).

## Caveat: requires full trust

One big caveat of this approach is that it doesn't work in partial trust, because launching msbuild requires full trust. This is not something that I think can be worked around easily, so I'd say it's an inherent limitation.

## Where can we take this?

Well, I'm not really sure yet, but it is certainly interesting to think about the possibilities of using this type of build model in ASP.NET.

Let me know if you think this is crazy or may have potential :)

