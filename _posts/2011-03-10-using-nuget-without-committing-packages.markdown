---
layout: post
title:  "Using NuGet without committing packages"
comments: true
categories: NuGet
---




**Update (8/16/2011): also check out this **[newer post](http://blog.davidebbo.com/2011/08/easy-way-to-set-up-nuget-to-restore.html)** that describes an easier way to set up this workflow.**

The current NuGet workflow has always been to commit the Packages folder into source control. The reasoning is that it matches what developers typically do when they don't have NuGet: they create a 'Lib' or 'ExternalDependencies' folder, dump binaries into there and commit them to source control to allow others to build.

While this has worked fine for some users, we have also heard from many that committing packages into source control is not what they want to do. When using a DVCS like Mercurial or Git, committing binaries can grow the repository size like crazy over time, making cloning more and more painful. In fact, this has been [one of the top requests](http://nuget.codeplex.com/workitem/165) on NuGet our issue tracker.

The good news is that NuGet now offers a workflow which goes a long way to solving this problem. It isn't 100% automated yet, but with some minimal pain you can set up your project to do this.

### Running 'nuget install' on a packages.config file

Earlier, I blogged about how you can [install NuGet packages from the command line](http://blog.davidebbo.com/2011/01/installing-nuget-packages-directly-from.html) by using NuGet.exe.

Get NuGet.exe from [here](http://nuget.codeplex.com/releases/view/58939) if you don't already have it, and run 'nuget -update' to self-update it.

This lets you install one package at a time, e.g.

```
D:\Mvc3Application>nuget install NHibernate -o Packages

```

As an aside, the -o flag is new and lets you specify where the package is installed.

But the big new thing is that you can now run it on a packages.config file. packages.config is a file that NuGet creates at the root of every project that has packages installed. So if you install the 'EFCodeFirst.Sample' package in your app, you'll find a packages.config next to the .csproj file, and it will contain:

```
<?xml version="1.0" encoding="utf-8"?>
<packages>
<package id="EFCodeFirst" version="0.8" />
<package id="EFCodeFirst.Sample" version="0.8" />
</packages>

```

So this holds all the information about what packages are needed for your project. Suppose you don't commit your Packages folder (which lives under the solution folder), and another developer clones your repository. They can now run:

```
D:\Mvc3Application>nuget i Mvc3Application\packages.config -o Packages
Successfully installed 'EFCodeFirst 0.8'.
Successfully installed 'EFCodeFirst.Sample 0.8'.

```

And the Packages will be restored! The other nice thing is that this command is smart enough not to do any expensive work if they are already installed, e.g.

```
D:\Mvc3Application>nuget i Mvc3Application\packages.config -o Packages
All packages listed in packages.config are already installed.

```

This completes very quickly with no network requests.

### Integrating package restore into msbuild

Integrating this into your build is a simple matter of adding a Pre-build event.

First, I would suggest committing nuget.exe into your solution, e.g. under a Tools folder. Once you do that, you can then add the following Pre-build event:

```
$(SolutionDir)Tools\nuget install $(ProjectDir)packages.config -o $(SolutionDir)Packages

```

Note how packages.config lives under the project folder while the Packages folder lives under the solution folder.

And that's it, you're done! Now each time you build, NuGet will first make sure that you have all the packages that you need, and will download anything that's missing from the live feed.

If your solution has multiple projects that use NuGet, add the same Pre-Build event to each project.

As an alternative, you can use an msbuild custom build target to achieve the same thing. Check out Danny Tuppeny's [post](http://blog.dantup.com/2011/05/setting-up-nuget-to-automatically-fetch-packages-when-deploying-to-appharbor-without-storing-binaries-in-source-control) for details on that. This worked better for him when using App Harbor.

### We want your feedback

This is new, so it's possible that it doesn't quite work perfectly in all cases. Please let us know how it works for you: bugs, feedback, suggestion. Thanks!

