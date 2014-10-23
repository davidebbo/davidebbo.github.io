---
layout: post
title:  "The right way to restore NuGet packages"
comments: true
tags: [NuGet]
---

Yesterday, I [tweeted](https://twitter.com/davidebbo/status/425493392475168768): 

> Every Time someone enables #nuget package restore on a solution, a kitten dies. Learn the new workflow!

It got a lot of RTs and Favs, but based on a number of comments, I think I may have caused some confusion, because it is in fact a confusing topic.

So first, let's get one thing out of the way: **I am not suggesting that you commit your NuGet packages into your git repo!** That is the worst thing you can do, and if you do that, you've moved on to killing unicorns.

The [NuGet docs](http://docs.nuget.org/docs/reference/package-restore) explain it all, but it's a little hard to read through, so I'll try to summarize the key points here. I'm keeping things concise, so please read that page for the complete story.

## The old way to do package restore

In the old way, you right click on your solution in VS and choose *Enable package restore*. This causes VS to modify your csproj files, and create .nuget folder containing nuget.exe and some other files.

After that, your packages get restored as part of msbuild when you build your project.

**Don't do this!** I hope the NuGet team will remove that option soon, and point people to...

## The Right way to do package restore

What the NuGet team is now recommending is both a lot cleaner and a lot simpler. In short, **you don't do anything special, and it just happens!** This is because NuGet now *always* restores packages before building in VS. So no changes whatsoever are needed on your files, which is beautiful!

Note: when building from the command line, you need to run 'nuget restore' yourself before msbuild. You could argue that scenario became harder than before, but it's for the greater good.


## Converting from the old way to the new way

The NuGet team has a [document](http://docs.nuget.org/docs/workflows/migrating-to-automatic-package-restore) that takes you step by step. In an ideal world, it would be automated, but going forward, if people stop using the Old Way on new projects, the issue will disappear over time.


## What if you have custom package sources?

All you need to do is create a [`NuGet.Config`](http://docs.nuget.org/docs/reference/nuget-config-settings) file next to your .sln file, containing:

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <packageSources>
        <add key="nuget.org" value="https://www.nuget.org/api/v2/" />
        <add key="aspnetwebstacknightlyrelease" value="https://www.myget.org/f/aspnetwebstacknightlyrelease/" />
      </packageSources>
    </configuration>

Note that if you have private package sources that you want to keep out of your repo, you can add them to %APPDATA%\NuGet\Nuget.config (see this [page](http://docs.nuget.org/docs/reference/nuget-config-file)) for details.
