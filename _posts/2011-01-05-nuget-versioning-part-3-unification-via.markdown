---
layout: post
title:  "NuGet versioning Part 3: unification via binding redirects"
comments: true
categories: [NuGet]
---


This is part 3 of the series on NuGet versioning.
- [NuGet versioning Part 1: taking on DLL Hell](http://blog.davidebbo.com/2011/01/nuget-versioning-part-1-taking-on-dll.html)
- [NuGet versioning Part 2: the core algorithm](http://blog.davidebbo.com/2011/01/nuget-versioning-part-2-core-algorithm.html)
- NuGet versioning Part 3: unification via binding redirects

In part 1 &amp; 2, we described the DLL hell problem, as well as the algorithm NuGet uses to achieve the best possible results.

Let's now look at how we can achieve runtime unification of assemblies using binding redirects.

## Strong names and binding redirects

Another important part of the story that we haven't yet discussed is assembly strong naming. When an assembly has a strong name, the binding to that assembly becomes very strict. That is, if assembly C depends on assembly X 2.0, it's not going to be happy with any other version of X, even if it's X 2.0.0.1.

**Note**: I'm now talking about assembly versions rather than package versions, but let's assume that they match, as will usually be the case.

Going back to our earlier sample in Part 2, what this means is that the package-level unification that we performed would have ended up breaking the app!

Recall that we had:

- A depends on X 1.1 (as in '>= 1.1')  
- B depends on X 1.2  
- C depends on X 2.0


And then (with the NuGet 1.1. twist), we ended up installing X 2.0.1.5, which doesn't match what A, B or C are looking for! If you then try to use A at runtime, and it itself tries to use X, you'll get a nasty error that looks like:

**Could not load file or assembly 'X, Version=1.1.0.0, Culture=neutral, PublicKeyToken=032d34d3e998f237' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference.**

While this looks scary, it's really just saying things the way they are: A was looking for X 1.1 and X 1.1 was nowhere to be found (since we have X 2.0.1.5).

The answer to this problem is to use [binding redirects](http://msdn.microsoft.com/en-us/library/twy1dw1e.aspx), which provide a simple and effective way of telling the runtime to bind to a different version than what an assembly was built against. e.g. in this case, you would add this to your web.config (or app.config):

```
<runtime>
<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
<dependentAssembly>
<assemblyIdentity name="X"
publicKeyToken="032d34d3e998f237" culture="neutral" />
<bindingRedirect
oldVersion="0.0.0.0-2.0.1.5" newVersion="2.0.1.5" />
</dependentAssembly>
</assemblyBinding>
</runtime>


```

This is basically telling the the runtime: “hey, if anyone asks you to load any version of X that's less than 2.0.1.5, please go ahead and load 2.0.1.5 instead”.

Once you do that, our A, B and C friends will all happily work against X 2.0.1.5. We have now achieved assembly-level unification to go along with our earlier package-level unification.

## Need some help writing those binding redirects?

You may be thinking that writing those binding redirects by hand is not as simple as I make it sound, and that it would be nice if you didn't have to worry about the details.

If so, you're in luck because NuGet comes with an Add-BindingRedirect command which will generate all the binding redirects for you!

It doesn't take any parameters. All you have to do is run it, and then check out your config file to see what was added (if you're curious). It's pretty smart about only adding things that are needed, so in simple situations that don't call for it, it will not make any changes.

**Note**: make sure that you build your app before running this command. We'll try to not make this required in the future, but for now, please remember to build first or it won't work correctly.

We are also considering ways to automate things even more such that you don't need to run any commands at all, and the binding redirects would get managed for you as you add packages. Not sure when we'll get there, but it should be feasible. in the meantime, just running Add-BindingRedirects is still a big help over hand writing those sections.

## Alternative to binding redirects

OpenWrap uses a different approach to solve that same issue: it modifies all the assemblies at install time to strip the strong name from them, hence allowing them to work with each other regardless of versions. Though it is a viable technique, we chose not to do this for a number of reasons.

The first is that it doesn't feel right to take an assembly that has been strongly named and signed by its author and turn it into something with no trace of the original signature. It may also have implications in some environments that only allow signed assemblies to run.

The second reason is that it violates one of our key design goals for NuGet, which is to **make it easy to do things that you could otherwise have done yourself without NuGet**. i.e. once NuGet has done its thing, it stays out of the way of your app, and you very much have a 'normal' app, not much distinguishable from what you would have if you had put it together without NuGet. But with the rewriting approach, you would end up with something that is quite different, making the installation process feel a little too 'magical' and not transparent enough.

The third reason may be the most important one: the rewriting approach locks you into using OW for everything. e.g. suppose that assembly A uses assembly X, and that you get both via OW. And now suppose that you get some other assembly B through some other mean (because there is no package for it yet), and that assembly also references X. You drop B.dll in bin and expect it to just work. But if X had been stripped of its strong named, B.dll would be broken (as it would fail to load the strong named X.dll). On the other hand, with the binding redirect approach, everything just works naturally, since no assemblies have been modified.

In the end, it comes down to NuGet and OW having rather different design goals, even though at a high level they share some similarities.

Update: it turns out that OW does not *yet* do this but it is planned for the next version. See Sebastien Lambla's new [post](http://codebetter.com/sebastienlambla/2011/01/05/strong-naming-assemblies-and-openwrap/?utm_source=twitterfeed&amp;utm_medium=twitter&amp;utm_campaign=Feed%3A+CodeBetter+%28CodeBetter.Com%29) on the topic for details.

## Conclusion

This completes this 3 part series on NuGet versioning, While there are still many areas that NuGet has not yet tackled, it has solid core approach to versioning and dependency management that it can build on.

