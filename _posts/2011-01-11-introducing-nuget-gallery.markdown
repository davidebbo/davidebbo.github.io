---
layout: post
title:  "Introducing the NuGet gallery"
comments: true
categories: [Orchard,NuGet]
---


Back in December, I [blogged](http://blogs.msdn.com/b/davidebb/archive/2010/12/25/an-update-on-the-nuget-package-submission-process.aspx) about how poor the NuGet package submission process was. You had to clone a HUGE repository that had all the other packages, add your package files to it, and submit a pull request. It was something that we had meant to last a couple weeks, and it lasted a few months, way overstaying its welcome.

The good news is that that process is now obsolete! Instead of have a brand new gallery site that lets authors publish packages very easily.

And the site it.. drums… [http://nuget.org](http://nuget.org/)!

## Who is this site for?

To set expectations, please note that this site is not feature complete yet, and is still rough in many ways. Eventually, it will be a place for both package authors and consumers, but in the short term, it's primarily useful to package authors.

So basically, this site provides a complete (and much better) replacement for the old package submission process, and at this point that is its main focus.

So if you are using NuGet from Visual Studio to install packages, you can probably ignore this site for now. It will be come interesting later, but it isn't now. You're certainly welcome to browse around it, but there is no point in creating an account now unless you have packages to submit.

## Getting started with the site

Here is what you need to do if you're a package author wanting the submit a package.

- Go to [http://nuget.org](http://nuget.org/), click Sign In, and Register Now  
- After registering, you'll get an email with a link you need to click (check your junk mail, it's probably there!).  
- An admin then needs to approve your account (see below for the reason behind that)  
- Once you're approved, you can just go to the Contribute tab and click Add New Package to upload your .nupkg files. They will be live on the feed instantly, though it may take a couple minutes for it to show up on the site itself.





## If you submitted packages with the old process

If you previously submitted packages using the old process, you will need to be given ownership of those packages in the new gallery before you can upload new versions.

Just ping @davidebbo on twitter and I can take care of that.

## Why the approval process?

Eventually, there won't be any approval process. The reason we chose to have one initially is that the site is still very young and we want to take it step by step. We will generally approve anyone that wants to get packages up there.

If you registered and don't see your account getting approved, please just ping me or Phil Haack on twitter (@davidebbo, @haacked).

## What, no Live ID or OpenID?

I know, this is really something we should be supporting, and we will later. We had to make that cut in order to get the gallery out sooner, as the current situation with package submission was simply not sustainable.

## Built on Orchard

The NuGet gallery was built using [Orchard](http://orchardproject.net/), which itself is still very young (1.0 release is around the corner). This could be one of the first real sites built using it, so it is both a great learning experience for the Orchard team, and a good showcase of the technology.

There were certainly a number of Orchard issues during development, but since the team is just down the hall from us, they took good care of things!

## Open Source

Just like NuGet itself, the gallery site was built as open source. Most of the development was done by [NimblePros](http://nimblepros.com/).

If you want to look through the sources, there are two CodePlex projects, one for the Orchard gallery site and one for the backend.

The gallery is at [http://orchardgallery.codeplex.com](http://orchardgallery.codeplex.com/), and the backend is at [http://galleryserver.codeplex.com/](http://galleryserver.codeplex.com/).

## How to report issues and give feedback

If you run into issues or want to give feedback about the site, feel free to start a discussion or file a bug on [http://orchardgallery.codeplex.com](http://orchardgallery.codeplex.com/).

