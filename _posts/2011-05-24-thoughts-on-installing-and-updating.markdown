---
layout: post
title:  "Thoughts on installing and updating NuGet packages outside of Visual Studio"
comments: true
categories: NuGet
---


One thing we hear occasionally from users is that it would be nice to be able to install NuGet packages from the command line (e.g. [this issue](http://nuget.codeplex.com/workitem/818), [that one](http://nuget.codeplex.com/workitem/902), and [this thread](http://nuget.codeplex.com/discussions/246942)). There are good reasons why this isn't supported today, which I will discuss in this post.

### What does installing a package even mean?

Before we go further, we need to be clear about exactly what we mean by 'installing a package'. The reason this needs to be clarified is that there are really two definitions, which can cause confusion.
- **Getting the bits onto the machine:** 'installing a NuGet package' is sometimes used to mean the act of getting the contents of the package onto your hard drive.  
- **Making a project use a package:** more commonly, it refers to not only downloading the package bits, but also 'applying' them to a project.


#1 is something that is fully supported today outside of Visual Studio using nuget.exe (see my [previous post](http://blog.davidebbo.com/2011/01/installing-nuget-packages-directly-from.html)). NuGet also supports [restoring packages outside of VS](http://blog.davidebbo.com/2011/03/using-nuget-without-committing-packages.html) so you don't have to commit them.

But for the sake of this post, I am strictly referring to #2, and that's what I mean any time I use the term 'installing a package' below.

Now that we have this out of the way, let's discuss why installing a package outside Visual Studio is non-trivial, as well as why it is in most cases not useful at all, although a case can be made for updating packages.

### What makes installing a package outside Visual Studio non-trivial

Installing a NuGet package into a project (e.g. a csproj file) is a rich operation which does a lot more than just copying files. Here is a rough list of what NuGet can do when you install a package from VS (whether using the Package Manager Console or the NuGet Dialog):
- Add references to assemblies contained in the package  
- Add references to framework assemblies in the GAC  
- Add content files to an app (e.g. JavaScript files, code files, …)  
- Add assembly binding redirects to deal with version incompatibilities  
- Perform config transformations, typically to add settings related to the package  
- Bring in tools that can then be run from Package Manager Console  
- Run PowerShell scripts which can do arbitrary things by automating the DTE object model







Now let's think about what it would take to perform those operations outside of VS.

The first 3 involve making modifications to the csproj file. When done within VS, it happens automatically by calling DTE methods, but outside of VS it would need to be done using custom parsing logic. While it's clearly possible, it needs to be done carefully to avoid corrupting the csproj file. e.g. a GAC reference should not be added if it's already there.

#4 to #6 should not be too different from doing it in VS.

#7 is basically impossible, since you cannot really 'fake' the DTE to let those script run.

So conceivably, with some good amount of work, we could support all scenarios except #7. It would be a little quirky as some packages would not fully work, but in many cases it would work.

But let's now discuss how useful it would be.

### Why installing packages outside of Visual Studio rarely makes sense

So let's say we had this feature and it fully worked. What would it let you do that you can't do today?

You could use the command line outside VS to install a Foo package in your project, but that in itself is rarely useful. e.g. suppose the package brings in a Foo.dll. You now have that assembly added as a reference in your project, but you don't have any code using it. You now need to go in VS to write code against that new assembly, so it would have been simpler to just add it from VS in the first place!

And that's generally the case for most packages: the logical step after installing them is to go to VS and actually use them, which mostly negates any benefits you may find by installing it outside of VS.

Admittedly, there are exceptions, like the Elmah package which is more or less 'ready to run' after you install it. But for the wide majority of packages, there is no direct 'install and run' workflow.

### What about package updates?

If package installs don't make sense outside of VS, what about package updates?

So you have this Foo packages that you installed from VS, but now you want to update it to a new versions from the command line. Does that make sense?

I think it does make a lot more sense than the install scenario, because by that point, you (presumably) already wrote some code that uses the package. So by updating it, you might get a newer Foo.dll with bug fixes, but all the code you wrote is still valid and ready to run against.

In particular, update could work well in the constrained scenario where the new version on the package just updates an assembly but doesn't do much else.

On the other hand, it would be hard to support in the general case, since in theory, the updated package can be completely different from the older one. e.g. suppose the new package contains some new install-time PowerShell scripts. We'd be right back with the same tough issues discussed above.

### Where do we go from here? You tell us!

My take is that we need to experiment with supporting package update outside on VS for at least a subset of scenarios. The big question is deciding how far this needs to go to reach sufficiently useful state.

The first step would be to start with the 'only the assembly changed' scenario, which is relatively simple, and probably is the 90+% case.

If you have some thoughts on this, we'd love to hear them! Would you use such feature, and would limiting it to updating assembly references be enough for your needs?

