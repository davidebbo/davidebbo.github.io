---
layout: post
title:  "One big feature social networks really need: Channels"
comments: true
tags: [TechThoughts]
---


Like many others, I have been playing around with Google+ to see what the new kid on the block has to offer. And it does have some good things going for it, with the concepts of Circles providing a pretty nice approach to privacy.

Unfortunately, it suffers from the same flaw that Facebook and Twitter have always had: **it makes the naïve assumption that when you follow someone, you want to hear everything they have to say**. In other words, it treats us as one-dimensional beings, which doesn't match the 'real world'.

This is something I have always found particularly painful on Twitter, both as a tweet consumer and a tweet producer.

As a consumer, I end up not following a bunch of interesting .NET folks because they're too 'noisy', meaning they tweet about a lot of unrelated things that I don't care about. I've tried to follow Scott Hanselman's philosophy and [let the river of crap wash over me](http://www.hanselman.com/blog/TwitterLetTheInformationWashOverYou.aspx), but it just didn't work for me. I guess I couldn't take the smell.

As a producer, I end up not tweeting many things I would want to say, because I know that a lot of my 2500 followers only care about the .NET side, and I don't want to add crap to their rivers. For instance, I follow tennis closely, but I'm not going to tweet super insightful things like “OMG, Federer lost!!”, because I know most followers don't care.

So to summarize, I'm missing out as a consumer, and repressed as a producer. Sad! :(

## Aren't Twitter hashtags the way to follow topics instead of users?

Twitter hashtags are an ugly hack over a weak platform, and don't do much to solve this.

First of all, as a producer, it makes no difference to my followers, since they will see my tweets no matter what hashtags they contain.

As a consumer, hashtags fail pretty badly for a number of reasons. First of all, many people don't use them correctly. They get misspelled, forgotten, and often conflict with unrelated things. But more importantly, they assume that you want to hear about that topic from everybody, while in many cases **I only want to hear what a selected set of users are saying about that topic**.

If I could set a search criteria for each user that I follow, I might be getting somewhere, but that's just not an option today. And even that would work poorly given the inconsistent use of hashtags.

## But don't Google+ Circles solve this issue?

No, not one bit! Circles are about privacy and nothing else. The issue I'm discussing here has nothing to do with privacy; it's about filtering of public information.

I see people saying that Google+ successfully merges what Facebook and Twitter are good at: connecting with friends and having a public voice. They are wrong! Let's put that to the test…

Let say I convince all my family to get on Google+ (a tough challenge, but bear with me). I add them to my 'family' circle and they do the same thing. We can share family things with great privacy; that's nice, and is where circles shine.

But now let's say I'm also using Google+ the way I use twitter today, and write a whole bunch of things about .NET.

What happens when my family members click on their 'family' circle? They're inundated with all that .NET stuff from me that they couldn't care less about! Their first reaction is that they want to go back to Facebook, where they don't see that 'work' stuff.

Now let's look at a second scenario: I want to publicly share various things about both .NET and tennis. They key word here is **publicly**. I don't want to have to add everyone who can read my tennis and .NET comments two circles, since I want it to be wide open. Circles are just not meant to solve this.

## The answer: Channels

One simple way to solve this is to add a concept called 'channels'. Here is how it would work:

First everyone can (optionally) define a list of channels. In my case, I might create channels called 'tech', 'tennis', and 'personal'. For each channel, you can write a one line 'advertisement' of what you generally discuss there. e.g. my tech channel might say 'stuff I work on, mostly related to .NET and NuGet'.

Then whenever you share something, you can choose whether it should go to everyone or just some channel. Note that when I say 'everyone' here, I really mean 'everyone that is allowed to see it'. Again, channels are not a privacy concept; they are orthogonal.

Finally, when you follow someone (i.e. add them to a circle), you get to choose whether you want the whole person, or only some of the channels. e.g. my mom would pick my 'personal' channel, while some .NET folks may choose 'tech', and others might leave it unfiltered and get it all (which would be the default, as it is today).

As an additional option, you could attach a channel to each circle. e.g. my 'family' circle would use to the 'personal' channel, so I don't have to think about it when I share from there. Note that this setting only applies to what I share. For each family member that I follow, I can still select what I want from their channels (which are likely not named the same as mine).

This may seem a bit complicated, but I don't think it would be in practice, because:

- Users coming from Facebook who only use it to connect to friends would not define any channels.  
- When you start following someone, you'd typically follow the whole person, as you do today. Then if you start getting too much noise from them, an easy-to-find option would allow you to tune it down. e.g. the context menu on my 'tennis' comment would offer “Don't show any more 'tennis' comments from this user”. Conceptually, this is similar to Facebook offering you to ignore Farmville entries from some users, and that's an easy concept to understand.



So it would not make the platform any less approachable to newbies, but the extra power would be readily available when needed.

## Good old blogs have had that forever

Interestingly, if you view 'things that you share' as 'blog posts', and 'following someone' as 'subscribing to their RSS feed', you find that the channel feature I describe here is almost identical to the concept of tags/labels in a blog.

e.g. You subscribe to [http://blog.davidebbo.com/](http://blog.davidebbo.com/) to get all my posts, and to [http://blog.davidebbo.com/search/label/NuGet](http://blog.davidebbo.com/search/label/NuGet) to only get my posts about NuGet.

So the basic concept is far from new, but for some reason the big social networks have not caught on to it.

## Will this feature ever be available?

Well, that's good question! My hope is that enough people want it that the big social networks will eventually want to implement something like it.

If I had to choose, I'd prefer Google+ to be the one offering this, since I think it has a model which lends itself to it best.

And if all else fails, I'll just have to start a new social network. Or not! :)

