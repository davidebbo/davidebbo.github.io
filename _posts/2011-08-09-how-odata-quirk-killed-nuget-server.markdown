---
layout: post
title:  "How an OData quirk slowed down NuGet, and how we fixed it"
comments: true
categories: [NuGet,OData]
---


Update: my terminology in this post is not quite correct. Whenever I refer to the server part of OData, I really mean to say 'WCF Data Services'. OData is the protocol, and WCF Data Services is the specific implementation. So the 'quirk' we ran into is a WCF Data Services thing and not an OData thing.

As you may know, NuGet uses an OData feed for its packages. Whenever you install packages, or search for packages from Visual Studio, it goes through this feed, which you can find at [http://packages.nuget.org/v1/FeedService.svc/Packages](http://packages.nuget.org/v1/FeedService.svc/Packages).

If you're a NuGet user, you may also have noticed that the perf of NuGet searches from Visual Studio had been quite bad in recent months. You'd go to the NuGet package dialog and type a search string, and it would take 10 or more seconds to give you results. Ouch! :(

It turns out that the perf issue was due to a nasty OData quirk that we've since worked around, and I thought it might be interesting to share this with others. I'm partly doing this as you might run into this yourself if you use OData, partly to poke a little fun at OData, and also to poke a little fun at ourselves, since we really should have caught that from day one.

## A whole stack of query abstractions

When you make an OData query from a .NET client, you go through a whole bunch of abstraction layers before a SQL query is made. Let's say for example that you're looking for packages that have the string 'T4MVC' in their description. It would roughly go though these stages:

First, in your .NET client, the OData client library would let you write something like:

{% highlight c# %}
var packages = context.Packages.Where(p => p.Description.Contain("T4MVC"));{% endhighlight %}

Second, this code gets translated by the OData client LINQ provider into a URL with a query string that looks like this:

```
?$filter=substringof('t4mvc',Description)

```

Third, this is processed by the OData server, which turns it back into a LINQ expressing, which in theory will look similar to what you had on the client, which was:

```
var packages = context.Packages.Where(
    p => p.Description.Contain("T4MVC"));
```

Of course, the 'context' here is a very different beast from what it was in step 1, but from a LINQ expression tree point of view, there shouldn't be much difference.

And finally, the Entity Framework LINQ provider turns this into a SQL query, with a WHERE clause that looks something like:

```
WHERE Description LIKE N'%T4MVC%'
```

And then it executes nice and fast (assuming a proper index), and all is well.

## When the abstractions break down

Unfortunately, that clean sequence was not going as planned, resulting is much less efficient queries, which started to get really slow as our package count started to get large (and we're already at over 7000 as of writing this post!).

So which of these steps went wrong? For us, it turned out to be the third one, where the OData server code was creating a very complex LINQ expression.

To understand why, let's first briefly discuss OData providers. When you write an OData DataService<T>, you actually have the choice between three types of providers:
- An **Entity Framework provider** which works directly over an EF ObjectContext 

- A **reflection provider** which works on an arbitrary context that exposes entity sets that are not tied to a specific database technology 

- A **custom provider**, which is something so hard to write that almost no one has ever done it (maybe a slight exaggeration, but not by much!)



Give that we're using EF, #1 seems like the obvious choice. Unfortunately, the EF provider is very inflexible, as it doesn't let you use any calculated properties on your entities. In other words, it only works if the only thing you want on your OData feed are fields that come straight from the database. So for most non-trivial apps, it's not a very usable option, and it wasn't for us (we have some calculated fields like ReportAbuseUrl).

So we ended up using the reflection provider, and wrapping the EF objects with our own objects which exposed whatever we wanted.

Functionally, this worked great, but what we didn't realize is that the use of the reflection provider causes OData to switch to a different LINQ expression tree generator which does 'crazy' things. Specifically, it makes the bad assumption that when you use the reflection provider, you must be using LINQ to object.

So it protects you by using some 'null propagation' logic which makes sure that when you write p.Description.Contain("T4MVC"), it won't blow up if the Description is ever null. It does this by inserting some conditional checks in the LINQ expression. This is very useful if you are in fact using LINQ to object, but it's a perf disaster if you are using LINQ to EF!

Now, when translated into SQL, what should have been the simple WHERE clause above was in fact becoming something like this:

```
WHERE  1 = ( CASE 
               WHEN ( Description LIKE N'%T4MVC%' ) THEN 
               CAST(1 AS BIT) 
               WHEN ( NOT ( Description LIKE N'%T4MVC%' ) ) THEN 
               CAST(0 AS BIT) 
             END ) 


```
which was running significantly slower. Note that in reality, we're querying for multiple fields at once, so the final SQL statement ended up being much scarier than this. I'm just using this simple case for illustration.And to make things worse, we learned that there was no way of turning off this behavior. What to do?
## 

## The solution: use some LINQ ninja skills to restore order

LINQ ninja [David Fowler](http://twitter.com/#!/davidfowl) found this an irresistible challenge, and came up with a fix is both crazy and brilliant: he wrote a custom LINQ provider that analyses the expression tree generated by the OData LINQ provider, searches for the unwanted conditional null-check pattern, and eliminates it before the expression gets handed out to the EF LINQ provider.

If you want to see the details of his fix, it's all on github, split into two projects:

QueryInterceptor ([https://github.com/davidfowl/QueryInterceptor](https://github.com/davidfowl/QueryInterceptor)) is a helper library that makes it easier to write this type of query modification code.

ODataNullPropagationVisitor ([https://github.com/davidfowl/ODataNullPropagationVisitor](https://github.com/davidfowl/ODataNullPropagationVisitor)) builds on QueryInterceptor and specifically targets the removal of the unwanted null check.

Naturally, these are available via NuGet (with the second depending on the first). After importing those packages, all that's left to do is add one small call to your IQueryable<T>, e.g.

```
query = query.WithoutNullPropagation();
```

and your expression trees will be given a gardener's special pruning :)

## Lesson learned: always check your SQL queries

Some might conclude that all those query abstractions are just too dangerous, and we should just be writing raw SQL instead, where this never would have happened. But I think that would be way too drastic, and I certainly wouldn't stop using abstractions because of this issue.

However, the wisdom we learned is that no matter what query abstractions you're using (LINQ, OData, or other), you should always run SQL query analyzer on your app to see what SQL statements get run in the end. If you see any queries that doesn't completely make sense based on what your app is doing, get to the bottom of it and address it!

Of course, this is really 'obvious' advice, and the fact that we never did that is certainly a bit embarrassing. Part of the problem is that our tiny NuGet team is mostly focused on the NuGet client, and that the server hasn't been getting enough love. But yes, these are just bad excuses, and in the end, we messed that one up. But now it's fixed :)

