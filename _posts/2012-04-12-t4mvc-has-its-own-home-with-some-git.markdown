---
layout: post
title:  "T4MVC has its own home, with some git love"
comments: true
categories: [T4MVC]
---


I have just moved T4MVC to a new CodePlex project, instead of it being part of the MvcContrib project. Its new home is [https://t4mvc.codeplex.com/](https://t4mvc.codeplex.com/).

If you're a T4MVC user, that should not make much difference except that there is now a new place to discuss it and file bugs. NuGet is still the place to go to get T4MVC!

Note that T4MVC is still part of the MvcContrib effort, even if it doesn't share the same source tree. Here are the reasons for the move.

## Reduce confusion

T4MVC is quite separate from the rest of MvcContrib, because it's just a T4 template, and not some code that's part of an assembly. Having the T4MVC files be in their own little island in the middle of a repo with many unrelated thing has been a bit of a barrier of entry for people wanting to make a quite contribution.

Also, since all MvcContrib bugs are files in the same place, there was always additional pain for me to filter T4MVC issues from unrelated ones.

Likewise, we'll now have our own [discussion forum](https://t4mvc.codeplex.com/discussions) that only focuses on T4MVC. Most users have been using StackOverflow for T4MVC support, and you can continue to do that if you prefer.

## Switch to git!

I've been increasingly using git over Mercurial (like everyone else it seems!), to the point where having to use Mercurial is becoming an annoyance. Since CodePlex now supports git, it was the perfect opportunity to switch to that!

