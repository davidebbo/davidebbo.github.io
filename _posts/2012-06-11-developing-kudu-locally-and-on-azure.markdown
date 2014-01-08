---
layout: post
title:  "Developing Kudu locally and on Azure"
comments: true
categories: [git,azure,kudu]
---


A few days ago, I [discussed](http://blog.davidebbo.com/2012/06/introducing-open-source-engine-behind.html) how the git support in Azure Web Sites was written as an Open Source project called [Kudu](https://github.com/projectkudu/kudu). This time, I'll give a few more details on how to run it, both locally and on Azure.

To be clear, you don't have to care about any of this if you just want to use git deployment to Azure. Just use it as is and it should work great!

This is for folks who are interested in modifying the git deployment engine itself, either to contribute some changes to the project, or just to play around with things.

## Running Kudu locally

First, you can see it all in action in this [3 minute screencast](http://www.youtube.com/watch?v=FmufYOz0KXI&amp;feature=plcp&amp;hd=1)!

Here are the basic steps to run Kudu locally. Note that this requires using IIS7, and will not work with IIS Express.

- Clone it from [https://github.com/projectkudu/kudu.git](https://github.com/projectkudu/kudu.git)
- In Visual Studio, open Kudu.sln. Important: VS needs to run as administrator!
- Set Kudu.Web as the Startup solution
- Ctrl-F5 to run
- You'll get an admin page that lets you create sites, and gives you the git URL for them
- Try git pushing a test project, and see it run!


**Important note**: the primary purpose of running Kudu locally is to make it easier to develop outside of Azure. Conceivably, you can take this non-Azure Kudu and host it on a VM, to have your own mini deployment server. However, it's missing a few features that would make it really usable there. e.g. it doesn't set up host names, and doesn't set up authentication. We would love to add these features, and welcome contributions!

## Running a private Kudu build on Azure

First, see it in action in this [5 minute screencast](http://www.youtube.com/watch?v=rcYXN6ACGi4&amp;feature=youtu.be&amp;hd=1).

This is the more crazy one. Suppose you want to make changes to the Kudu service, and make it even more awesome. :) You can make these changes locally and test them outside of Azure, per the previous section.

But wouldn't it be great if you could actually use your latest Kudu bits in Azure itself? Turns out you can, using a special hook that we put in for the exact purpose.

Here are the basic steps:

- As above, clone Kudu from [https://github.com/projectkudu/kudu.git](https://github.com/projectkudu/kudu.git)
- Make whatever changes you want to the sources
- Run build.cmd at the root of the repo. This creates an artifacts\debug\KuduService folder than contains the built Kudu bits.
- Go to the [Azure portal](https://manage.windowsazure.com/) and create a new app, then enable git.
- [Use FTP to connect to your files](https://github.com/projectkudu/kudu/wiki/Accessing-files-via-ftp), and copy the KuduService folder at the root. Here is [what the file structure looks like](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure) in there.
- In the Portal, go to the Configure tab, and add an 'app setting' with name/value USE_PRIVATE_KUDU=1 (and **don't forget to hit Save at the bottom!**).


And you're done! If you now do a git push to your Azure site, you are now using your very own git engine instead of the one that comes with Azure. How cool is that? :)

**Important notes:**

- Doing this only affects this one site. If you have multiple sites where you want to use your private Kudu bits, you'll need to set up each of them the same way.
- It probably goes without saying, but once you are running your own git deployment engine, you're in unsupported territory, so don't call product support if something doesn't work! However, the Kudu team will always be happy to talk to you on [JabbR](http://jabbr.net/#/rooms/kudu), or our [MSDN forum](http://social.msdn.microsoft.com/Forums/en-US/azuregit/threads), or on [github](https://github.com/projectkudu/kudu). :)


