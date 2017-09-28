---
layout: post
title:  "Unit test your MVC views using Razor Generator"
comments: true
tags: [MVC,NuGet,ASP.NET,Razor,RazorGenerator]
---


Click [here](http://blog.davidebbo.com/tag/#RazorGenerator) to find all the posts relating to the Razor Generator

A few days ago, I [blogged](http://blog.davidebbo.com/2011/06/precompile-your-mvc-views-using.html) about how you can use [Razor Generator](http://visualstudiogallery.msdn.microsoft.com/1f6ec6ff-e89b-4c47-8e79-d2d68df894ec) to precompile your MVC Razor views. 
In this post, I will demonstrate how you can then unit test your precompiled views. Note that this is still very much experimental, so at this point the primary goal is to get feedback on the concept.
## Simple walkthrough to unit test views

After installing RazorGenerator, create an MVC 3 Razor app, using the 'Internet Application' template and including the unit test project.

In the previous post, we used precompiled views in a different library, so this time let's keep them in the MVC project to show something different.
First, use NuGet to install the RazorGenerator.Mvc package in your MVC project.Then, as in the previous post, set the custom tool on Views\Home\Index.cshtml to 'RazorGenerator', causing Index.cs to be generated under it.But now, let's do something new and use NuGet again to add the RazorGenerator.Testing package to the unit test project (**not** to the MVC app!).And that's all it takes to get set up! Now we can write a unit test for our precompiled Index.cshtml view. e.g. create a Views\HomeViewsTest.cs (in the unit test project): 
{% highlight c# %}
using HtmlAgilityPack;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using MvcApplication2.Views.Home;
using RazorGenerator.Testing;

namespace MvcApplication1.Tests.Views {
    [TestClass]
    public class HomeViewsTest {
        [TestMethod]
        public void Index() {
            // Instantiate the view directly. This is made possible by
            // the fact that we precompiled it
            var view = new Index();

            // Set up the data that needs to be accessed by the view
            view.ViewBag.Message = "Testing";

            // Render it in an HtmlAgilityPack HtmlDocument. Note that
            // you can pass a 'model' object here if your view needs one.
            // Generally, what you do here is similar to how a controller
            //action sets up data for its view.
            HtmlDocument doc = view.RenderAsHtml();

            // Use the HtmlAgilityPack object model to verify the view.
            // Here, we simply check that the first <h2> tag contains
            // what we put in view.ViewBag.Message
            HtmlNode node = doc.DocumentNode.Element("h2");
            Assert.AreEqual("Testing", node.InnerHtml.Trim());
        }
    }
}

{% endhighlight %}

## A few notes about unit testing views

Unit testing views in ASP.NET MVC is something that was very tricky to do before, due to the fact that the views are normally compiled at runtime. But the use of the Razor Generator makes it possible to directly instantiate view classes and unit test them.
Now the big question, is whether unit testing views is desirable. Some people have expressed concerns that it would be a bit fragile due to the changing nature of the HTML output.My take here is that while it would be a bad idea to try to compare the entire HTML output, the test can be made pretty solid by selectively comparing some interesting fragments, as in the sample above.That being said, I have not tried this is a real app, so there is still much to learn about how this will all play out. This is just a first step! 
## What about partial views?

When designing this view testing framework, we took the approach that we wanted to focus on the output of just one view at a time. Hence, if a view calls @Html.Partial(…) to render a sub-view, we don't let the sub-view render itself, and instead just render a token to mark where the sub-view would be.
This seemed more true to the nature of what a unit test should be, compared to letting the whole composite page render itself, which would be more of a functional test (plus there were some tough challenged to making it work). 
## Where do we go from here?

Well, it'll be interesting to hear what people think about the general idea. We're interested in two types of feedback.
First, what do you think about the overall concept of unit testing views using this approach. Second, please report bugs that you run into to [https://github.com/RazorGenerator/RazorGenerator](https://github.com/RazorGenerator/RazorGenerator). At this point, I expect it to be a bit buggy and probably blow up on some complex views. Treat it as a proof of concept! :)
