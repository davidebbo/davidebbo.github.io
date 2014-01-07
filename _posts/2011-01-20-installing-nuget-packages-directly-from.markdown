---
layout: post
title:  "Installing NuGet packages directly from the command line"
categories: NuGet
---


Most of the coverage around NuGet revolves against its clean integration to Visual Studio, which makes adding references to packages as easy as adding references to local assemblies.  While this is indeed a key scenario, it is important to note that the core of NuGet is completely decoupled from Visual Studio, and was designed with that goal from day 1.

If we look back at the early days of NuGet, it was in many ways inspired by the 'Nu' project (which members have since joined NuGet).  What Nu had was a solid command line driven experience to bring down .NET bits to your machine.  In their case, it was based on Ruby Gems, but that is an implementation details.  Take a look at Rob Reynolds's original [screencast](http://www.youtube.com/watch?v=IvxAa4XURss) to see what the Nu experience was about.

While we have been planning all along to provide the same experience with NuGet (in addition to the VS experience of course), it's something that had somewhat fallen off the radar, and it just had not been done.  This was unfortunate, because we already had all the plumbing to make it happen, and all it needed was about 10 lines of code to expose this!

So I'm happy to say that we have now filled this little hole by implementing a new 'install' command in our NuGet.exe command line tool.  Using it couldn't be any easier, and I'll walk you through an example.

### Where do I get NuGet.exe?

You first need to get NuGet.exe.  This is the same tool that package authors have been using to create packages and upload them to the [http://nuget.org](http://nuget.org/) gallery.

The easiest way to get it is to [download it](http://nuget.codeplex.com/releases/view/58939) from CodePlex.

You can also obtain it via NuGet itself by installing the package name NuGet.CommandLine (using Visual Studio).

### How do I run it?

The best way to demonstrate it is to just show a sample session.

```
D:\>md \Test

D:\>cd \Test

D:\Test>nuget list nhi
FluentNHibernate 1.1.0.694
FluentNHibernate 1.1.1.694
NHibernate 2.1.2.4000
NHibernate 3.0.0.2001
NHibernate 3.0.0.3001
NHibernate 3.0.0.4000
NHibernate.Linq 1.0
NHWebConsole 0.2
SolrNet.NHibernate 0.3.0

D:\Test>nuget install NHibernate
'Iesi.Collections (≥ 1.0.1)' not installed. Attempting to retrieve dependency from source...
Done.
'Antlr (≥ 3.1.3.42154)' not installed. Attempting to retrieve dependency from source...
Done.
'Castle.Core (≥ 2.5.1)' not installed. Attempting to retrieve dependency from source...
Done.
Successfully installed 'Iesi.Collections 1.0.1'.
Successfully installed 'Antlr 3.1.3.42154'.
Successfully installed 'Castle.Core 2.5.2'.
Successfully installed 'NHibernate 3.0.0.4000'.

D:\Test>tree
Folder PATH listing
Volume serial number is 26FF-2C8A
D:.
├───Antlr.3.1.3.42154
│   └───lib
├───Castle.Core.2.5.2
│   └───lib
│       ├───NET35
│       ├───NET40ClientProfile
│       ├───SL3
│       └───SL4
├───Iesi.Collections.1.0.1
│   └───lib
└───NHibernate.3.0.0.4000
└───lib

D:\Test>dir Antlr.3.1.3.42154\lib
Volume in drive D has no label.
Volume Serial Number is 26FF-2C8A

Directory of D:\Test\Antlr.3.1.3.42154\lib

01/20/2011  05:06 PM           117,760 Antlr3.Runtime.dll

```

### 

### Why would you want to use this instead of the Visual Studio integration?

For most users, the Visual Studio integration will be the right choices.  But suppose you want to work much more 'manually', and not deal with VS or even with a .csproj file.  e.g. all you want is to bring down nhibernate.dll so you can write some code against it, and compile it manually using 'csc /r:nhibernate.dll MyCode.cs'.

In this scenario, you just want NuGet to download the assemblies for you, and leave the rest to you.  It still saves you a lot of time by letting you easily download the bits and all their dependencies, but it doesn't force you into a development model that may not be want you want.

So I don't think it's a feature that the majority of users will use, but it is important to have it for those who need it.

