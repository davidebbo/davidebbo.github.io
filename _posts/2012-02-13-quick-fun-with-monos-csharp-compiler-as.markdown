---
layout: post
title:  "Quick fun with Mono's CSharp compiler as a service"
comments: true
tags: [Mono]
---


Mono comes with a really cool [CSharp compiler as a service](http://www.mono-project.com/CSharp_Compiler). The only problem is that no one seems to know about it!

I think the main reason for this is that anything related to Mono causes a fair bit of confusion to all the people who are not familiar with it. And that certainly includes myself, as I know very little about it besides what I'm discussing in this post!

Talking to various people, the general misconceptions are:

- Mono only runs on Linux  
- Even if it runs on Windows, it doesn't use the CLR, so I can't use it  
- Mono is for strange people :)




And while that may be true for some aspects of Mono, it certainly isn't for Mono.CSharp.dll. In fact, it's a totally 'normal' library that you can use in your very 'normal' C# projects in Visual Studio.

The next hurdle is that it's not all that easy to just get Mono.CSharp.dll. You have to either install an 80MB setup from [here](http://www.go-mono.com/mono-downloads/download.html), or get a big .tar.gz file with lots of other things from [here](http://mono.ximian.com/daily/monocharge-latest.tar.gz). And a lot of people on Windows don't like dealing with tar.gz files (hint: use [7zip](http://www.7-zip.org/)).

Now the good news: after chatting with [Miguel de Icaza](https://twitter.com/#!/migueldeicaza) on Twitter, I put Mono.CSharp.dll [on NuGet](https://nuget.org/packages/Mono.CSharp), making it totally trivial to use from VS. There goes that hurdle. (note: I'm the package owner for now, until some Miguel-blessed dev claims it).

## Try Mono.CSharp in under 5 minutes

Just open VS and create a Console app, and add a NuGet package reference to Mono.CSharp. That takes a whole 30 seconds. And I'll re-emphasize that there is nothing 'Mono' about this Console app. It's just plain vanilla.

Now write some basic code to use the compiler. It all revolves around the Evaluator class. Here is the sample code I used ([GitHub](https://github.com/davidebbo/MonoCompilerDemo)). It's quick and dirty with poor error handling, as the focus is to just demonstrate the basic calls that make things work:

{% highlight c# %}
using System;
using System.IO;
using Mono.CSharp;

namespace MonoCompilerDemo
{
    public interface IFoo { string Bar(string s); }

    class Program
    {
        static void Main(string[] args)
        {
            var evaluator = new Evaluator(
                new CompilerSettings(),
                new Report(new ConsoleReportPrinter()));

            // Make it reference our own assembly so it can use IFoo
            evaluator.ReferenceAssembly(typeof(IFoo).Assembly);

            // Feed it some code
            evaluator.Compile(
                @"
    public class Foo : MonoCompilerDemo.IFoo
    {
        public string Bar(string s) { return s.ToUpper(); }
    }");

            for (; ; )
            {
                string line = Console.ReadLine();
                if (line == null) break;

                object result;
                bool result_set;
                evaluator.Evaluate(line, out result, out result_set);
                if (result_set) Console.WriteLine(result);
            }
        }
    }
}

{% endhighlight %}

It feeds it some starter code and start a REPL look to evaluate expressions. e.g. run it and try this. You type the first two, and the 3rd is output:

```
MonoCompilerDemo.IFoo foo = new Foo();
foo.Bar("Hello Mono.CSharp");
HELLO MONO.CSHARP

```

You get the idea!

## What about Roslyn?

I [blogged](http://blog.davidebbo.com/2011/10/using-roslyn-to-implement-mvc-razor.html) a few months back about using Roslyn to implement an MVC Razor view engine. I'm far from a Roslyn expert, and frankly haven't done much with it since that post. From what I read, Roslyn has the potential to enable some very compelling scenarios in the future.

But there is one major argument right now in favor of using the Mono compiler: it's pretty much feature complete **today**, while Roslyn is [not even close](http://social.msdn.microsoft.com/Forums/en-US/roslyn/thread/f5adeaf0-49d0-42dc-861b-0f6ffd731825). Totally understandable given that it's a CTP, and is only meant to give an early taste of the feature.

So anyway, I still know close to nothing about Mono, but if I need to dynamically compile some pieces of C# in a 'normal' non-Mono project, I know that Mono.CSharp is not far away!

