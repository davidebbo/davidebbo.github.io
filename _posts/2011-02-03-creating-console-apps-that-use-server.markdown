---
layout: post
title:  "Creating Console apps that use the Server Profile"
comments: true
categories: [Visual Studio]
---


When you create a Console app in Visual Studio 2010, it gets create in a way that it targets the “.NET Framework 4 Client Profile”.  What that means is that it can't use any ASP.NET component, which for ASP.NET developers is rather useless.

The standard way to fix this is to go to the Project's properties and change the Target Framework:

![image](https://lh6.googleusercontent.com/_jySMpScpTXc/TUsw2ZxV7wI/AAAAAAAAAUc/YV46vka-on4/image_thumb%5B1%5D.png?imgmax=800)

You then get an alert telling you that your project needs to be reloaded:

![image](https://lh4.googleusercontent.com/_jySMpScpTXc/TUsw3Mj4lbI/AAAAAAAAAUk/B0m39qb-4sk/image_thumb%5B3%5D.png?imgmax=800)

And once you click yes, you can be on your way to other greatness.

While this works, it's frankly painful when you have to do this many times a day.  To make things worse, if you forget to do it, you often get strange failures which don't make it obvious what the issue is, leading to frustration or worse.

I have no idea who was behind the decision to make the default be the client profile, but I'll go on record saying that it was a dumb idea! :)

### Fix this permanently using a custom project template

Luckily, it's pretty easy to fix this by using a custom VS Project Template.  [@luhmann](http://twitter.com/#!/Luhmann) sent one to me, so I didn't even have to write it :)

Here is what you need to do:
- Go to this folder: %USERPROFILE%\Documents\Visual Studio 2010\Templates\ProjectTemplates\Visual C#\
- Under that, create a 'Windows' folder if you don't already have one (you probably don't)
- Download the custom template from [here](https://docs.google.com/uc?id=0B9LFjrvVZR24MmZmMzNlZTUtNTU0Zi00M2FiLTk1ODUtMTY0ODk2MjdiMDA4&amp;export=download&amp;hl=en), and save it into that Windows folder (but don't unzip it!).

Now when you need to create a C# Console app, you'll see a new entry from the custom template:

![image](http://lh6.ggpht.com/_jySMpScpTXc/TUsw36kDU2I/AAAAAAAAAUs/9MT-W2blWkk/image_thumb%5B9%5D.png?imgmax=800)

If you use that, your console app won't be using the evil Client Template, which will lead to greater happiness.

Note that if you really wanted, you could replace the default template by that one, but I like seeing both entries side by side as a reminder of what's going on.  And who knows, some day I might just want to use the Client Template!

