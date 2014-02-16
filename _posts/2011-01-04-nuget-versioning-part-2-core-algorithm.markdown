---
layout: post
title:  "NuGet versioning Part 2: the core algorithm"
comments: true
tags: [NuGet]
---


This is part 2 of the series on NuGet versioning.
- [NuGet versioning Part 1: taking on DLL Hell](http://blog.davidebbo.com/2011/01/nuget-versioning-part-1-taking-on-dll.html)
- NuGet versioning Part 2: the core algorithm  
- [NuGet versioning Part 3: unification via binding redirects](http://blog.davidebbo.com/2011/01/nuget-versioning-part-3-unification-via.html)<!--EndFragment-->

In part 1, we described the two sides of DLL hell, as well as how assembly Unification is superior to Side by Side.

Let's now dive into the algorithm that NuGet uses to deal with versioning.

## Package vs. Assembly

It should be noted that at the top level, NuGet deals with Packages rather than assemblies. Those packages in turn can bring in zero or more assemblies. The assembly versions may or may not match the package version, though is most cases they do.

The following discussion on versioning is referring primarily to Package versions, though the reasoning applies equally well to DLL versions (and essentially falls out of it).

## How NuGet specifies dependency versions

The [NuGet syntax](http://nuget.codeplex.com/wikipage?title=Version%20Range%20Specification) for specifying package dependency versions borrows from the [Maven specification](http://maven.apache.org/enforcer/enforcer-rules/versionRanges.html), which itself borrows from [mathematical intervals](http://en.wikipedia.org/wiki/Interval_(mathematics)). e.g. when component A depends on component X, it can specify the version of X that it needs in two different ways (in the [.nuspec file](http://nuget.codeplex.com/wikipage?title=Nuspec%20Format)):
- A range, which can look like [1.0,3.0), meaning 1.0 or greater, but strictly less than 3.0 (so up to 2.*). See spec above from more examples.  
- A simple version string, like “1.0”: this means “1.0 or greater”

Your first reaction may be that #2 is counter intuitive, and should instead mean “exactly 1.0”. The reason it means “greater or equal'” is that as things turn out, this is what should be used most of the time in order to get the best behavior, i.e. in order to avoid both of the extremes of DLL hell mentioned above. This reason will soon become clear.

## The version selection algorithm

Having a version range is only half of the puzzle. The other half is to be able to pick the best version among all the candidates that are available.

Let's look at a simple example to illustrate this:

- A depends on X 1.1 (meaning '>= 1.1' as discussed above)  
- B depends on X 1.2  
- C depends on X 2.0  
- X has versions 1.0, 1.1, 1.2, 2.0, 3.0 and 4.0 available


The version resolution used by NuGet is to **always pick the lowest version of a dependency that fits in the range** (a small exception to this is mentioned further down). So let's see what will happen in various scenarios:

- If you just install A, you'll get X 1.1  
- If you just install B, you'll get X 1.2  
- If you just install C, you'll get X 2.0  
- If you first install A, then B then C  

- You'll initially get X 1.1 when you install A  

- X will be updated to 1.2 when you install B  

- X will be updated to 2.0 when you install C




The crucial point here is that even though A and B state that they can use any version of X, they are **not getting forced into using anything higher than necessary**.

It may very well be that A does not work with much higher versions of X like 3.0 and 4.0, and in that sense you can say that the specified range is 'wrong'. But that is simply not relevant unless you are in a situation where you must use those higher versions due to a different component in the same app depending on those higher versions.

If we had instead specified exact versions, we would not have allowed anything to work together, even though the components may very well be backward compatible up to a point. That is one of the extremes of DLL hell discussed in Part 1: inability to find a version that everyone can work with.

Likewise, if the algorithm had picked the highest version in range, we would have ended up with X 4.0 in all scenarios. That is the other extreme of DLL hell: a newly released component breaks scenarios that were working before.

The simple algorithm NuGet uses does a great job of walking the fine line between those two extremes, always doing the safest thing that it can while not artificially disallowing scenarios. As an aside, that is essentially the same as what Maven does (in the Java world), and this has worked well for them.

## When an upper bound makes sense

In most cases, simply specifying a minimum version is the way to go, as illustrated above. This does not imply that upper bounds shouldn't be specified in some cases.

In fact, an upper bound should be specified whenever a component is **known** not to work past a certain version of a dependency.

e.g. in our example, suppose that A is known not to work with X 2.0 or greater. It would then be fine to specify the range as [1.1,2.0). And in the scenario above, when you try to install C after installing A, you'd get a failure to install. i.e. A and C simply cannot be used in the same app. Clearly, this is a bit better than allowing the install to happen and then having things break at runtime.

But the key thing here is that the incompatibility has to be **known** before such range is used. e.g. if at the time A is written, X 2.0 doesn't even exist, it would be wrong to set a range of [1.1,2.0).

I know, it may feel like the right defensive thing to do not to allow running against something that doesn't yet exists, but doing so creates many more issues than it solves in the long run.

The rule of thumb here is that a dependency version is “**innocent until proven guilty**”, and not the other way around.

## Backward compatibility is in the eye of the consumer

A subtle yet very important point is that simply knowing that version 2.0 of X has some breaking changes over version 1.2 doesn't mean all that much.

e.g. you may be tempted to say that if B uses X 1.2 and X 2.0 has some breaking changes over 1.2, then B should never use 2.0. But in reality, doing so is too conservative, and causes the second form of DLL hell (inability use some components together, and general lack of flexibility).

The more important question to ask is whether X 2.0 has breaking changes **that affect B**. B may very well be using a small subset of the API's and be unaffected by the breaking change. So in this situation, you should not jump to the conclusion that you need a [1.2,2.0) range.

Again, “innocent until proven guilty”. Or maybe I should say “give (DLL) peace a chance”, or “if it ain't broke, don't prevent it”. Or maybe I should stop there ;)

Credits to [Louis DeJardin](http://twitter.com/#!/loudej) on convincing me of this key point.

## NuGet 1.1 twist

Earlier, I mentioned that NuGet's algorithm was to “always pick the lowest version of a dependency that fits in the range”. That is true of NuGet 1.0, but in 1.1 or later, we added a small twist to this, which is to always move up to the highest build/revision. Confused? An example will make it clear.

Let's take our example above, but now say that X's available versions are 1.0, 1.1, 1.2, 2.0, **2.0.0.1**, **2.0.1.0**, **2.0.1.5**, 3.0, **3.0.1** and 4.0.

When installing A, B and C, with NuGet 1.0 we would end up with X 2.0. But with 1.1, we'd get version 2.0.1.5. The reason this is important is that the last two numbers are typically non-breaking bug fixes, and the assumption is that you are always better off picking them over an older build with the same Major/Minor version (i.e. the same first two numbers).

## A few words on Semantic Versioning

[Semantic Versioning](http://semver.org/) (SemVer) describes a way for authors to define versions in a way that they have a consistent semantic. In a nutshell, semantic versions look like X.Y.Z (Major.Minor.Patch), such that:

- A change in X is a breaking change  
- A change in Y adds functionality but is non-breaking  
- A change in Z represents a bug fix


The use of this versioning scheme is not widely adopted today, but I think it would be beneficial if component authors (and NuGet package authors) followed it more.

Currently, the only case where NuGet makes some use of SemVer is with the “1.1 twist” described above, which causes it to move up to a slightly newer version that has 'bug fixes'.

Technically, if all components actually honored SemVer, we could always safely move from 1.0 to 1.1, as it would be guaranteed to be a non-breaking upgrade. But in practice, this would not work well today given how a change in Minor version (Y) does often contain breaking changes.

It is also worth noting that the NuGet algorithm described above makes this mostly unnecessary, because there is no reason to use 1.1 if the component asks for 1.0. Unless of course some other component needs 1.1, in which case we would use it.

In part 3, we will discuss how NuGet makes use of CLR binding redirects to achieve assembly unification.

