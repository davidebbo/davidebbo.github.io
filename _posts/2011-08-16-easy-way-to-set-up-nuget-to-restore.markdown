---
layout: post
title:  "The easy way to set up NuGet to restore packages"
comments: true
categories: [NuGet]
---


Note (12/22/2011): in NuGet 1.6 or later, this feature is built in, so you no longer need to use the NuGetPowerTools. Just right click on the Solution and choose 'Enable NuGet Package Restore'.

A few months ago, I described a workflow that lets you [use NuGet without committing the packages to source control](http://blog.davidebbo.com/2011/03/using-nuget-without-committing-packages.html). This has been a very popular workflow, and generally works quite well.

The down side is that it's a bit painful to set up: you have to get NuGet.exe and add it to your tree, then you have to add a pre-build event for every project that uses NuGet.

The good news is that the ever-resourceful [David Fowler](http://twitter.com/#!/davidfowl) has come up with a much easier way to set that up, using his [NuGetPowerTools](https://github.com/davidfowl/NuGetPowerTools) package. Here is the way to do it:

Let's assume that you have a solution that is either already using NuGet, or planning to use it, and that you want to set up the no-commit workflow.

Now, you just need to go to the Package Manager Console and run a couple commands:

```
PM> Install-Package NuGetPowerTools
Successfully installed 'NuGetPowerTools 0.28'.

**********************************************************************************
INSTRUCTIONS
**********************************************************************************
- To enable building a package from a project use the Enable-PackageBuild command
- To enable restoring packages on build use the Enable-PackageRestore command.
- When using one of the above commands, a .nuget folder will been added to your
solution root. Make sure you check it in!
- For for information, see https://github.com/davidfowl/NuGetPowerTools
**********************************************************************************


PM> Enable-PackageRestore
Attempting to resolve dependency 'NuGet.CommandLine (≥ 1.4)'.
Successfully installed 'NuGet.CommandLine 1.4.20615.182'.
Successfully installed 'NuGet.Build 0.16'.

Copying nuget.exe and msbuild scripts to D:\Code\StarterApps\Mvc3Application\.nuget
Successfully uninstalled 'NuGet.Build 0.16'.
Successfully uninstalled 'NuGet.CommandLine 1.4.20615.182'.

Don't forget to commit the .nuget folder
Updated 'Mvc3Application' to use 'NuGet.targets'
Enabled package restore for Mvc3Application

```

And you're done! So basically, the first command installs a NuGet package which brings in some helpful commands, and the second one runs one of those commands.

After doing this, you'll notice a new .nuget folder under your solution, containing nuget.exe plus a couple msbuild target files. **Make sure you commit that folder to source control!** You'll also find a few changes in your csproj files to trigger the restore functionality when you build.

I have now become a strong supporter of the don't commit packages workflow, and if you're going to use it, this is the way to do it!

