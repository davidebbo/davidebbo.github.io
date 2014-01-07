---
layout: post
title:  "Introducing Kudu: the Open Source engine behind 'git push azure master'"
comments: true
categories: git azure kudu
---


Yesterday, ScottGu unveiled [the new Azure](http://weblogs.asp.net/scottgu/archive/2012/06/07/meet-the-new-windows-azure.aspx), which brings a whole list of exciting changes to the platform.

One of the most exciting new features is the ability to deploy Web Sites to Azure using git. Scott's post covers that, and I also did a [screencast](http://www.youtube.com/watch?v=72SAHWUHnzA&amp;hd=1) on that topic.

One part that has not yet been discussed is that the engine that powers this feature was developed as an Open Source project from the first line. The project is code named Kudu and can be found at [https://github.com/projectkudu/kudu](https://github.com/projectkudu/kudu). Kudu is a member of the the [Outercurve Foundation](http://www.outercurve.org/), and is released under the Apache License 2.0 (the same as [NuGet](http://nuget.codeplex.com)).

This project is actually not tied to Azure, and can run standalone on any machine. In that mode, you can push project and have them run in your local IIS.

### So why is this project interesting to you?

There are a few reasons that you may be interested in this project.

The first is that it's a good place to file bugs that you run into when you git push your project to Azure. You can also use our [forum](http://social.msdn.microsoft.com/Forums/en-US/azuregit) to discuss things.

The second reason is that the associated [wiki](https://github.com/projectkudu/kudu/wiki) contains lots of useful info about it. Well, at this time there isn't all that much there, but the idea is that we'll grow it as we go forward. And of course, wiki contributions are welcome!

And finally, you may be interested in contributing to the project, as we do accept contributions!

