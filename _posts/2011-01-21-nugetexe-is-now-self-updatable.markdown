---
layout: post
title:  "NuGet.exe is now self-updatable"
comments: true
categories: NuGet
---


Yesterday, I [blogged](http://blog.davidebbo.com/2011/01/installing-nuget-packages-directly-from.html) about how the NuGet command line tool can now be used to bring down packages without using VS.

Another cool new trick that it just gained is the ability to update itself.  What that means is that after you get the tool on your machine (e.g. get the latest from [here](http://nuget.codeplex.com/releases/view/58939)), keeping it up to date becomes super easy.

I'll demonstrate how it works by example.  First, let's run nuget.exe with no params just to see what version we have:

```
D:\>nuget
NuGet Version: 1.1.2120.136
usage: NuGet <command> [args] [options]
Type 'NuGet help <command>' for help on a specific command.
etc...
```

We're running 1.1.2120.136.  Now let's check for updates:

```
D:\>nuget update
Checking for updates from http://go.microsoft.com/fwlink/?LinkID=206669.
Currently running NuGet.exe v1.1.2120.136.
Updating NuGet.exe to 1.1.2121.140.
Update successful.
```

And now let's make sure we're running the new one:

```
D:\>nuget
NuGet Version: 1.1.2121.140
usage: NuGet <command> [args] [options]
Type 'NuGet help <command>' for help on a specific command.
etc...

```

And just like that, we're now running the newer build!

### How is the update performed

Being a package manager, it's pretty natural for NuGet to be able to do that, as NuGet.exe is itself a package in its own feed!  The package is named NuGet.CommandLine.

To perform the in-place update, nuget.exe simply renames itself to nuget.exe.old, and downloads the new one as nuget.exe.  The old file can then be deleted, or if for whatever reason you're not happy with the newer build, you can simply delete it and rename nuget.exe.old back into nuget.exe.

### What about updates to the NuGet Visual Studio add-in?

Just a final note in case you're wondering why update is done this way for nuget.exe, but not for the NuGet VS integration.  Since the VS tooling is a standard extension, it gets an update story 'for free' via the VS Extension Manager.  In VS, just go into Tools / Extension Manager and go to the Updates tab, which will tell you if there are updates available to any of the extensions you have installed.

