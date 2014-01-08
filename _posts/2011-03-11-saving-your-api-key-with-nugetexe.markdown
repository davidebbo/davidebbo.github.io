---
layout: post
title:  "Saving your API key with nuget.exe"
comments: true
categories: [NuGet]
---


While you can push NuGet packages from [http://nuget.org/](http://nuget.org/), it is often more convenient to do it straight from the command line using nuget.exe.

Phil did a nice [post](http://haacked.com/archive/2011/01/12/uploading-packages-to-the-nuget-gallery.aspx) on how that works, which you should read first if you have not done this before.

The one pain point about this technique is that you need to find your API key every time you push a package. I have had to go to [http://nuget.org/Contribute/MyAccount](http://nuget.org/Contribute/MyAccount) each time to locate my key, copied it and pasted it on the command line. It gets old quickly! :)

The good news is that the newest version of nuget.exe (get it [here](http://nuget.codeplex.com/releases/view/58939)) lets you save it once and for all! Credit goes to [Matthew Osborn](http://twitter.com/#!/osbornm) for this new feature.

Here is how it works.

## Saving your key

First, you run the new SetAPIKey command, e.g.

```
D:\test>nuget SetApiKey 78a53314-c2c0-45c6-9d92-795b2096ae6c
The API Key '78a53314-c2c0-45c6-9d92-795b2096ae6c' was saved for the source 'http://go.microsoft.com/fwlink/?LinkID=207106'.


```

This encrypts the key and saves it in a config file under your %APPDATA% folder. e.g. mine ends up in C:\Users\davidebb\AppData\Roaming\NuGet\NuGet.Config. This file contains:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<apikeys>
  <add key="http://go.microsoft.com/fwlink/?LinkID=207106" value="AQAAANCMnd8BFdERjHoAwE/Cl+sBAAAAnMGkdu4+rkqpSdQUWwjfIgAAAAACAAAAAAADZgAAwAAAABAAAAA5gG4wxeb8Vn4X0Y0p//OvAAAAAASAAACgAAAAEAAAAF/llublBpBgL9lSFaE9/A0oAAAAC4NVHflYsUU5UgVgOq+h3t1jwY6l2BEji6Td4F0lvxsZcZ73L2m6BRQAAABJ0TZLKdIYStn8DWawbtzdo3mrKg==" />
</apikeys>
</configuration>


```

Note that the key is saved per server URL, with the server defaulting to nuget.org (you can pass -src to change that).

## Using the saved key

Once you have done this one-time step, pushing packages becomes a breeze, as the only thing you need to pass is your package file. e.g.

```
D:\test>nuget push DavidTest.1.0.nupkg
Publishing DavidTest 1.0 to the live feed...
Your package was published to the feed.


```

Likewise, if you want to delete a package, you'd do:

```
D:\test>nuget delete -noprompt DavidTest 1.0
Deleting DavidTest 1.0 from the server.
DavidTest 1.0 was deleted from the server

```

Hopefully this will make your NuGet package management experience a little bit easier!

