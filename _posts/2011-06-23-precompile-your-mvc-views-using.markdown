---
layout: post
title:  "Precompile your MVC Razor views using RazorGenerator"
comments: true
tags: [MVC,ASP.NET,Razor,RazorGenerator]
---


Click [here](http://blog.davidebbo.com/search/label/RazorGenerator) to find all the posts relating to the Razor Generator A while back, I [blogged](http://blogs.msdn.com/b/davidebb/archive/2010/10/27/turn-your-razor-helpers-into-reusable-libraries.aspx) about a single file generator I wrote that can precompile Razor helpers. A bit later, [Chris van de Steeg](http://twitter.com/#!/csteeg) started from that code base and extended it to support precompiling MVC views (see [his post](http://www.chrisvandesteeg.nl/2010/11/22/embedding-pre-compiled-razor-views-in-your-dll/)).

On my side, this project stayed dormant for a little while, but recently we started extending it to support a number of interesting new scenarios, including precompiling MVC views (albeit with a slightly different approach from Chris's).

Most of the new code was written by [Pranav Krishnamoorthy](http://twitter.com/#!/pranav_km), a dev on the ASP.NET team. Many thanks to him for moving this project forward. 

## Installing the generator

It's on the VS extension gallery, so install it from there. It's called “Razor Generator” (not to be confused with “Razor Single File Generator for MVC”, which is Chris').

![image](http://lh5.ggpht.com/-jkOfjQOV26M/TgLpLIbuETI/AAAAAAAAAYo/B2r3nk0WFKo/image_thumb%25255B1%25255D.png?imgmax=800)

## 

## Walkthrough to precompile MVC views

You can use it to precompile MVC views either in a separate library or in the MVC project itself. I'll demonstrate the separate library case, as it's a bit more interesting.

To begin with, create a new MVC 3 app using Razor (and the 'Internet Application' template). Then add a new class library project to the solution (e.g. call it MyPrecompiledViews), and add a reference to it from the MVC project.


Update (4/26/2012): the best approach is to actually create an MVC project for that library, instead of a library project. You'll never actually run it as an Mvc app, but the fact that it comes with the right set of config files allows intellisense and other things to work a lot better than in a library project. See [http://razorgenerator.codeplex.com/](http://razorgenerator.codeplex.com/) for latest info.
Now the fun part begins: using NuGet, install the RazorGenerator.Mvc package into your class library. This adds a number of things to the project:

- A reference to RazorGenerator.Mvc.dll, which contains the view engine  
- Logic to register the view engine using WebActivator (in App_Start\PrecompiledMvcViewEngineStart.cs).  
- Two web.config files that are there to make intellisense work while you author your views (they're not used at runtime)  
- A sample view, which you can later remove





Let's take a closer look at that sample view:

![image](http://lh5.ggpht.com/-MQpzV2zjSZs/TgLpLkUkiBI/AAAAAAAAAYw/ji5bVXveX0M/image_thumb%25255B4%25255D.png?imgmax=800)

![image](http://lh3.ggpht.com/-Py4u8iBx64w/TgLpMGkQSzI/AAAAAAAAAY4/1C5hEvLy0pA/image_thumb%25255B8%25255D.png?imgmax=800)

Notice that it has a Custom Tool set to RazorGenerator, which causes it to generate a .cs file underneath itself (thanks to the generator we installed earlier).

This is just a sample, so now let's move the Views\Home\Index.cshtml from the MVC project to the same folder in the class library (you can press Shift during the drag/drop to make it a move). Then set the generator to RazorGenerator as in test.cshtml. You'll now get an Index.cs nested under Index.cshtml.

**And that's it you're done!** You can now run your app, and it will be using the precompiled version of Home\Index.cshtml. 

## Why would you want to do that?

One reason to do this is to **avoid any runtime hit** when your site starts, since there is nothing left to compile at runtime. This can be significant in sites with many views.

Also, you no longer need to deploy the cshtml files at all, resulting in a **smaller deployment file set**.

Another cool benefit is that it gives you the ability to **unit test your views**, which has always been something very difficult with the standard runtime compilation model. I'll cover that in more details in a future post. 

## Generating files at design time vs. build time

The way the generation works is very similar to T4 templates you have you project. The generation happens as soon as you save the file. You can also force it to regenerate by right clicking on the .cshtml file and choosing Run Custom Tool.

Generally, the guidance is to commit those generated files along with the cshtml file, the same way that you commit all your 'hand-written' source files. If you do that, everything will run just fine in an automated build environment.

Another reason to commit the generated files is that it allows you to write code against them with full VS intellisense. e.g. if you use this technique to write Razor helpers that you want to call from other views, you really want VS to know about the generated file at design time. Ditto if you want to write unit tests against your views.

That being said, if you really want to postpone the generation until build time, we're working on an MsBuild task that will do that. For now, you can find it by getting the RazorGenerator sources on CodePlex. 

## If you want to help or report issues

This project is hosted on [http://razorgenerator.codeplex.com/](http://razorgenerator.codeplex.com/) under the Apache 2.0 Source license, so feel free to contribute! You can also use CodePlex to discuss and report issues.

