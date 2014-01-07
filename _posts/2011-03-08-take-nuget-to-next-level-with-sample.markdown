---
layout: post
title:  "Take NuGet to the next level with sample packages"
comments: true
categories: [NuGet]
---


[NuGet](http://nuget.org/) has drastically simplified the process of getting .NET libraries into your projects. What used to be an error prone and painful process has become as simple as adding an assembly reference.

While it has solved an important part of the developer workflow, it has the potential to also solve another key piece of the puzzle: helping user learn to use libraries.

### I found these cool packages, but now what?

There are tons of cool packages available on NuGet today, and the number is growing daily. I've heard of a number of users who go down the list and install all kind of packages into their projects to try them out. But if you're not familiar with a library, how do you get started with it?

As an example to illustrate the discussion, let's take the nifty little Clay package written by the Orchard guys. Say you have installed it into your project and want to start using it. Here is what you might do:

- The NuGet dialog gives you a link to the 'project URL'. Typically, it's a link to where the project is hosted on CodePlex/BitBucket/github, and indeed this one takes you to [http://clay.codeplex.com/](http://clay.codeplex.com/).
- Once you're there, you try clicking on the Documentation tab. Unfortunately, many projects don't have much there. But here it at least has a pointer to [Bertrand](http://twitter.com/#!/bleroy)'s blog posts on the topic. So you now go to his [post](http://weblogs.asp.net/bleroy/archive/2010/08/18/clay-malleable-c-dynamic-objects-part-2.aspx).
- You read through it, and after a while, you can piece together enough bits and pieces to know what it's about and start using it into your code.


I took Clay as an example, but this is a fairly typical experience. The fact is that a lot of knowledge about immature (yet useful) projects only exists in 'blog post series' rather than in any formal documentation. Not ideal, but that's how things happen.

### NuGet to the rescue with Sample Packages

Luckily, there is a simple and effective solution to this problem: use NuGet to distribute basic samples that get your users on the right path with less pain.

So to illustrate this post, I went ahead and created one such package for Clay: **Clay.Sample**. This package depends on Clay, such that installing it also installs Clay (as well as other things Clay depends on, like Castle).

It's a 'source only' package, meaning that it doesn't contain any binaries of its own. So let's go ahead and try it in a brand new Console app (and change it NOT to use the client profile). Go in NuGet's 'Add Library Reference' dialog and search for Clay. You'll get this:

![image](http://lh3.ggpht.com/_jySMpScpTXc/TXXwc4POocI/AAAAAAAAAU0/k-Gr7AnCT5E/image_thumb%5B7%5D.png?imgmax=800)

After you install it, your project will look like this:

![image](http://lh4.ggpht.com/_jySMpScpTXc/TXXwdiVcNvI/AAAAAAAAAU8/zDQcBUpmZHw/image_thumb%5B13%5D.png?imgmax=800)

First, note how you got all the expected references to Clay and to its dependencies: Castle.* and log4net.

But more interestingly, it also brought in a ClaySamples source file under Samples\Clay. It contains a number of Clay samples, which I shamelessly copied from Bertrand's post. Here is one example:

{% highlight c# %}
public static void AnonymousObject() {
   dynamic New = new ClayFactory();

   var person = New.Person(new {
       FirstName = "Louis",
       LastName = "Dejardin"
   });

   Console.WriteLine("{0} {1}", person.FirstName, person.LastName);
}
{% endhighlight %}

There are about 10 such samples in there, which demonstrate everything that the post discusses. Now go to your Console Main and make a call to a method that runs all the samples:

{% highlight c# %}
class Program {
   static void Main(string[] args) {
       Samples.Clay.ClaySamples.RunAll();
   }
}

{% endhighlight %}

While there is nothing in there that's not in the blog post, the big advantage is that you can trivially get it into your project via NuGet, and you can then directly run/debug the samples without having to piece them together.

Of course, the blog post (or documentation) may still be worth reading for extra insight. But you may find that the samples give you all you need for now, and save the deeper reading for later.

### Call to packages authors: write Sample packages!

I think this type of packages can have a huge impact on developer productivity. But for that to actually happen, those packages need to be created! And while I created the one for Clay, I am not volunteering to create all the sample packages :) Clearly, the best person to do that is the author of the package, though anyone who knows it well enough can certainly do it as well.

So if you own a NuGet package, please try to take on that task. It's super easy, and your users will thank you for it!

### Conventions, conventions, conventions

I recently blogged about using the [App_Start convention](http://blog.davidebbo.com/2011/02/appstart-folder-convention-for-nuget.html) for WebActivator startup code and got a great response, with almost all WebActivator users converting their existing packages to use this.

The situation here is quite similar, and calls for a similar convention, which is what I showed above. In a nutshell:

- If your package is named Blah, call the sample package Blah.Sample. If you want multiple sample packages, you can call them Blah.Sample.Something and Blah.Sample.SomethingElse.
- Make your Blah.Sample package dependent on Blah.
- Within that package, just include source files. Place those file under the Samples\Blah. You can have one or more, and call them whatever you think make sense.
- The code on there is up to you, but the general idea to to include whatever you think will help the user get started. Try to make the sample code **easily runnable** without too much extra setup. This may be harder for some packages, but do your best :)


### Creating the package

Taking Clay as an example, here is the structure of the files before packing them into a nupkg:

{% highlight c# %}
├  Clay.Sample.nuspec
└──Content
└──Samples
 └──Clay
    └  ClaySamples.cs.pp{% endhighlight %}

So there are just two files, the nuspec and the preprocessed sample file. Here is the nuspec:

```
<?xml version="1.0"?>
<package xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<metadata xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
<id>Clay.Sample</id>
<version>1.0</version>
<authors>Outercurve Foundation</authors>
<owners>Outercurve Foundation</owners>
<licenseUrl>http://www.opensource.org/licenses/ms-pl</licenseUrl>
<projectUrl>http://clay.codeplex.com</projectUrl>
<requireLicenseAcceptance>false</requireLicenseAcceptance>
<description>This package contains samples that demonstrate the use of the Clay library.</description>
<language>en-US</language>
<dependencies>
<dependency id="Clay" version="1.0" />
</dependencies>
</metadata>
</package>

```

The interesting parts here are the package Id, the description, and the dependency on Clay.

Then ClaySamples.cs.pp is a normal source file, except for a tiny bit of preprocessing for the namespace, e.g.

{% highlight c# %}
using System;
using ClaySharp;

namespace $rootnamespace$.Samples.Clay {
   public static class ClaySamples {
      // Sample code here
   }
}

{% endhighlight %}
And that's it! Once you have that, just run 'nuget pack' from the folder with the nuspec, and you'll have a sample package ready to be pushed to the feed.
