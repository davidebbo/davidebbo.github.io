---
layout: post
title:  "NuGet versioning Part 1: taking on DLL Hell"
comments: true
tags: [NuGet]
---


NuGet makes it easier than ever to get all kind of libraries into your .NET apps. While that is its most obvious benefit, NuGet also helps tremendously with managing dependencies and versioning, which can normally be a complicated process.

In this multipart series, I will cover the following topics:
- NuGet versioning Part 1: taking on DLL Hell  
- [NuGet versioning Part 2: the core algorithm](http://blog.davidebbo.com/2011/01/nuget-versioning-part-2-core-algorithm.html)
- [NuGet versioning Part 3: unification via binding redirects](http://blog.davidebbo.com/2011/01/nuget-versioning-part-3-unification-via.html)



Before going too deep into the NuGet behavior, let's step back and look at the problem we're tackling: the infamous DLL hell.

## The two extremes of DLL hell

I have seen the term 'DLL hell' used to describe situations happening at both ends of the spectrum.

The more common usage refers to what occurs when the **versioning policy is too loose** (or non-existent). This is the classic old 16-bit Windows issue where a DLL gets updated system wide, and everything on the system that was using the old version is now using the newer one, sometimes causing breaks due to incompatibilities.

At the other end of the spectrum, there is what happens when the **versioning policy to too tight**, as is often the case with the GAC. Here, you can lose the ability to use components A and B at the same time, because they each want to use a different version of component X, and the system won't let those be unified.

So in one case we get in trouble because apps get broken, while in the other we get in trouble because apps can't use the components that they need.

## BIN deployment limits the scope of the issue

To begin with, it's worth emphasizing that NuGet never installs assemblies machine wide. In particular, it will never put anything into your GAC. Instead, all assemblies are bin deployed, which means they only affects the current app. This reduces the scope of the issue because it moves the concern from a potential machine wide DLL hell to a potential application wide DLL hell. So even if things are not done correctly, you can still get bad things to happen within an app, but you'll never mess up a different app.

So NuGet's focus is on doing the right thing **within this one app**.

## Unification versus Side by Side: a clear choice

When dealing with situations where two different versions of a DLL appear to be needed, there are two possible approaches.

The first approach is **unification**, which consists in picking one version of that DLL and using it for the entire app. This can require the use of Binding Redirects, as we will discuss later in the series.

The second approach is to allow both versions of the DLL to run **Side by Side**. e.g. A could be using X v1.0 while at the same time B could be using X v1.1.

NuGet always uses unification, as Side by Side is evil for several reasons. First, using Side by Side is difficult for a practical reason: the bin folder can only contain one file named X.dll. But even if you get around this (e.g. by going crazy with AssemblyResolve events), it is likely to get you in trouble because many assemblies don't expect this. e.g. an error logging component expects to be the only one there, and would end up fighting with its evil twin if they ever came to run in the same app domain at the same time.

So let's settle this one: **two versions of an assembly should never be loaded in the same app**. I'm not saying that there aren't any situations where it may legitimately arise, but for the most part, it's best to not go there, 

In part 2, we will discuss how NuGet deals with package and assembly versioning.

