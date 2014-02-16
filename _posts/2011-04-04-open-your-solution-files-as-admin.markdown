---
layout: post
title:  "Open your solution files as admin"
comments: true
tags: [Visual Studio]
---


When you run under UAC (User Account Control), nothing runs as admin by default, and that's a good thing! But sometimes, you do need to run some things as administrator.

There are a few well known ways of doing this. You can right click on an EXE and choose 'Run As Admin'. Or if you have the app pinned on your taskbar, you can Ctrl-Shift click it to run as admin. If you don't know those tricks, you should learn them as they often come handy.

However, there is one common scenario for which there is no well documented technique: how do you launch a program as admin **from a data file**? Taking a particularly interesting example, how do you launch Visual Studio as admin from a .sln file?

First, you try the obvious and right click it, hoping to see the familiar 'Run As Administrator' item. But no luck there:

![image](http://lh3.ggpht.com/_jySMpScpTXc/TZlxGJV5gMI/AAAAAAAAAVg/5hrIH2-SO2g/image_thumb%5B1%5D.png?imgmax=800)

While this at first appears hopeless, it turns that there is a way to do this by adding some simple things to your registry.

The general technique is explained [here](http://www.howtogeek.com/howto/windows-vista/add-run-as-administrator-to-any-file-type-in-windows-vista/) (thanks to [@meligy](http://twitter.com/#!/Meligy) for pointing me to it). The post describes how to do it for any file type, but I can save you a bit of time by giving you the reg change you need to make (and it's not scary!):

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\VisualStudio.Launcher.sln\Shell\runas\Command]
@="\"C:\\Program Files (x86)\\Common Files\\Microsoft Shared\\MSEnv\\VSLauncher.exe\" \"%1\""


```

Just save that in a foo.reg file somewhere and run it. After you do that, right clicking on a .sln file will look like this:

![image](http://lh3.ggpht.com/_jySMpScpTXc/TZlxGlLnnoI/AAAAAAAAAVo/7bDFLs5luQk/image_thumb%5B8%5D.png?imgmax=800)

And that's it, just what we wanted!

**Final note**: my reg file above is hard coded to “C:\\Program Files (x86)”, which won't work on all systems so you may need to adjust things. I tried to change it to use the ProgramFiles(x86) env variable but I couldn't make that work in the registry. Seems default values can't be REG_EXPAND_SZ? Let me know if you know how to do this!

