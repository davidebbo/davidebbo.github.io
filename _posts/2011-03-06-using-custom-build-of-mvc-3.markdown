---
layout: post
title:  "Using a custom build of MVC 3"
comments: true
categories: [MVC,ASP.NET]
---


**Disclaimer**: running a custom build of MVC 3 is not recommended in most scenarios. Please run against the official MVC 3 bits unless you really cannot. Be aware that using custom builds will make it harder to use 3rd party libraries built against the official bits (you'll need to rebuild those libraries).

One nice thing about ASP.NET MVC is that you [can get the full sources on CodePlex](http://aspnet.codeplex.com/releases/view/58781) and compile them yourself.

Once you copy the sources locally, you can open the WebRuntime solution in VS 2010 and build it. As an aside, note that this solution also contains the ASP.NET WebPages sources, since MVC3 makes you of some of that logic (mostly for Razor support).

So this lets you modify the sources and build everything. However, one thing that makes the use of the resulting assemblies a bit tricky is that unlike the official bits, the bits produced by this solution are unsigned.

Let's take this step by step.

## Step 1: Create a new MVC 3 application

So let's start by creating a new MVC 3 Web Application using the regular project template that come from installing MVC 3.

This gives you a working app, but obviously at this point you're still using the official MVC 3 bits.

## Step 2: Reference your custom assemblies

The next step is to reference your custom MVC assemblies. Start by removing the System.Web.Mvc and System.Web.WebPages references. Instead, reference the version you've built of those same assemblies, which you'll find under mvc3\bin\Debug (from the root of the WebRuntime solution).

Once you do that, your MVC project will build fine. However, if you try running it, you'll get some scary looking runtime compilation error. Something like:

```
CS0433: The type 'System.Web.Mvc.WebViewPage<TModel>' exists in both 'c:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Web.Mvc\v4.0_3.0.0.0__31bf3856ad364e35\System.Web.Mvc.dll' and 'c:\Users\David\AppData\Local\Temp\Temporary ASP.NET Files\root\d305385c\948d4291\assembly\dl3\ef116fd6\5f110ce1_44dccb01\System.Web.Mvc.DLL'

```

The reason this happens is that while you've changed the **project references** to point to your assembly, the two web.config files that come with the project template are still pointing to the official assemblies left and right. Which leads us to…

## Step 3: Fix up your web.config files

The project comes with two web.config files, and they each contain all kind of references to the official assemblies (which are strong name). e.g. in the root web.config, you'll find:

```
<add assembly="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
<add assembly="System.Web.WebPages, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />

```
Luckily, the fix is as simple as yanking the strong name from a few assemblies. This is easily done using a project-wide search/replace. Specifically, do the following three replacements:

1. Replace all instances of (**excluding the quotes!**)

“System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35”

by

“System.Web.Mvc”

2. Replace all instances of

“System.Web.WebPages, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35”

by

“System.Web.WebPages”

3. Replace all instances of

“System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35”

by

“System.Web.WebPages.Razor”

And that should be it. Your app will now be up and running against your custom MVC bits.

