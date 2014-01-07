---
layout: post
title:  "Using Roslyn to implement an MVC Razor view engine"
comments: true
categories: [MVC,ASP.NET,Razor]
---


**Update 12/29/2011: the Roslyn CTP is now available on NuGet, so it's no longer necessary to install it before running this sample!**

****

**Note**: the code for this view engine sample is [on Github](https://github.com/davidebbo/RoslynRazorViewEngine).

The C# team has just announced the public availability of the first Roslyn CTP. See their post [here](http://blogs.msdn.com/b/visualstudio/archive/2011/10/19/introducing-the-microsoft-roslyn-ctp.aspx), and download it from [here](http://www.microsoft.com/download/en/details.aspx?id=27746&amp;utm_source=feedburner&amp;utm_medium=twitter&amp;utm_campaign=Feed%3A+MicrosoftDownloadCenter+%28Microsoft+Download+Center%29#tm). I really hope they can make it available on NuGet soon, but right now it's not there, so you'll have to run their setup. Sorry!

As you've probably heard from various conferences earlier this year, Roslyn offers a compiler as a service for C# and VB. Since we do a lot of compilation in ASP.NET land, I figured I'd play around with trying write an MVC view engine that uses it instead of the standard compilation path.

**Word of warning**: the Roslyn CTP is still very rough and is missing a lot of key features, like dynamic, anonymous types, indexers and using statements (get the full list [here](http://social.msdn.microsoft.com/Forums/en-US/roslyn/thread/f5adeaf0-49d0-42dc-861b-0f6ffd731825)). So while I did get something working, the language limitations prevent it from being useful in any real scenario. This is just an exercise to see how far we can get. Lower your expectations! :)

### Why would we want to do this

When you have a standard MVC project, compilation happens at two different levels:

- Your Controllers, Models, and most of your C# code get compiled by msbuild (or Visual Studio) into a single assembly which ends up in the 'bin' folder  
- All the Views (whether .aspx or .cshtml) get compiled dynamically at runtime by ASP.NET.


One drawback of compiling views at runtime is that it's pretty slow. And since it's slow, ASP.NET tries really hard to save assemblies to disk so it can reuse them across AppDomain cycles. Those assemblies all go under the infamous 'Temporary ASP.NET Files' folder. There is a huge amount of complexity to make this work, with settings like batching which can either help or hurt depending on the situation.

One thing I've been working on to avoid this dynamic compilation is [RazorGenerator](http://razorgenerator.codeplex.com/), which lets you precompile your views into the same assembly as your controllers. This works quite nicely, but it does have one big drawback: you can't just update a view and have it get picked up at runtime. Instead, you need to rebuild using msbuild (or VS), just like you would when you change a controller file.

What would be nice is to be able to support dynamic compilation of the views, but with a much lighter system then what the standard ASP.NET Build Manager provides. Enter Roslyn!

### Compile views using Roslyn: fast and lightweight

The main reason that the standard build manager is pretty slow is that it goes through CodeDom, which launching csc.exe for every compilation. csc.exe is actually very fast at compiling C# code, but the fact that we have to pay for the csc process startup time each time we compile anything ends up making things slow.

By contrast, Roslyn gives us an API to compile code in memory, without ever having to launch another process, making things much faster. In fact, it is so fast that the incentive that we had to preserve compiled assembly in 'Temporary ASP.NET Files' mostly disappears.

Instead, we can take a much simpler approach: whenever we need to compile a view, we just compile it on the fly in memory using Roslyn, and then cache it for the lifetime of the AppDomain. But we never need to cache it to disk, and generally don't use the disk at all.

In preliminary tests, I have measured the perf of compiling pages using Roslyn to be more than 50 times faster than doing it via CodeDom. So it's looking quite promising!

So to summarize, the benefits of using Roslyn to implement a view engine are:

- Fast dynamic compilation  
- No need to cache assemblies to disk, leading to a much simpler and lighter weight system. 
- New shiny thing! :)




### More detail about the code

The code for my sample view engine is on Github ([https://github.com/davidebbo/RoslynRazorViewEngine](https://github.com/davidebbo/RoslynRazorViewEngine)), so I'll mostly let you check it out there. All the interesting code is in RoslynRazorViewEngine.cs.

Here are the main steps that it goes through to turn a Razor file into an Assembly:
- First it uses the Razor Engine to generate a CodeCompileUnit from the Razor file.
- It then uses CodeDom to turn the CodeCompileUnit into C# source code. Note that we only use CodeDom as a code generator here, and not to actually compile anything.
- We then use Roslyn to compile the course code into a byte[]. That byte array is basically an in memory copy of what would normally be a .dll file.
- Finally, we call Assembly.Load to load that byte[] into a runtime Assembly.

### How restrictive are the limitations in the Roslyn CTP?

As I mentioned above, there are lots of limitations, which make this little more than a proof of concept.

To begin with, it doesn't support dynamic, which MVC uses pretty heavily. By default, MVC views extend WebViewPage<dynamic>, so I had to add '@model object' at the top of my test view to get around that.

Then there is ViewBag, which is also dynamic, and allows writing things like '@ViewBag.Message'. I tried replacing that by '@ViewData["Message"]', only to find out that indexers were not supported either. Duh!

And then it doesn't support anonymous objects, which MVC uses quite a bit...

So don't even think of trying to use this for anything real at this time. Still, the approach feels pretty sound, and whenever Roslyn becomes more feature complete, I have good hope that it can help us improve the ASP.NET compilation system.

