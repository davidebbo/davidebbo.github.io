---
layout: post
title:  "Converting my old blog"
comments: true
tags: Blogging
---

Yesterday, I [blogged](/2014/01/moving-to-github-pages.html) about my reasons to move away from Blogger, and why I picked GitHub pages to replace it. Today, I'll discuss some of what I went through to port my old blog over.

I'm not going to go in great details about parts that have been discussed everywhere, like using GitHub pages.


## Porting the comments to Disqus

I thought this would be the hardest part, but it turned out to be quite easy. I already had a Disqus account that I needed earlier in order to comment on other sites. All I had to do was add my own site to it, and tell it to import all comments from Blogger. One little OAuth workflow later, all my comments were on Disqus!


## Getting Jekyll installed

First, I had to get Jekyll running on my machine, which is quite a bit more painful on Windows than on Mac/Linux. I found a good [post](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/) that guided me through all the steps, and there sure are quite a few of them!

Even though I have it working, it produces some weird errors/warnings that appear to be harmless:

    D:\Code\Github\davidebbo.github.io>jekyll build
    Configuration file: D:/Code/Github/davidebbo.github.io/_config.yml
                Source: D:/Code/Github/davidebbo.github.io
           Destination: D:/Code/Github/davidebbo.github.io/_site
          Generating... C:/Ruby200-x64/lib/ruby/gems/2.0.0/gems/posix-spawn-0.3.8/lib/posix/spawn.rb:162: warning: cannot close fd before spawn
    'which' is not recognized as an internal or external command,
    operable program or batch file.
    done.

So `cannot close fd before spawn`, and missing `which` (even though I have that on my PATH from git). Whatever, it still works so I'm ignoring that.


## Porting the posts

That's where things got nasty. The [Jekyll import page](http://import.jekyllrb.com/) makes it look really easy: install the `jekyll-import` gem and profit.

Sadly, I just couldn't get that damn gem to install, and after some investigation, I concluded that it's just busted on Windows (see this [thread](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#comment-1188702765)).

If I had any common sense, I would have switched to using my old MacBook Air, which probably would have worked. But I didn't have the Mac with me at that time, and I didn't want to wait. So I did the usual dumb thing that devs do: I wrote my own conversion tool from scratch!

To make things more interesting, I had decided ahead of time that I didn't want to keep my old posts as HTML (even though Jekyll supports that), and instead wanted everything as Markdown. Just because.

So here is the [tool](https://github.com/davidebbo-test/BlogConverter) I wrote for the job.

**Warning**: it's all dirty, hacky, and comment free. I wrote it for this one-time purpose, it did the job, and now I'm done with it. I had fun doing it, too! If someone finds a need for it, be my guest, but expect roughness :)

High level, here is what it does:

- reads the big XML file that I downloaded from blogger, which contains all the posts and comments (of course, I don't care about the comments at this point).
- extracts all the Post's metadata out of it: title, date, tags, ...
- gets each post's content, and convert it for HTML to Markdown. I used the brilliant [Html Agility Pack](https://htmlagilitypack.codeplex.com/) to parse the HTML (I'm not crazy enough to do this from scratch). And then I just went through the tags to convert them to Markdown.
- writes out all the Jekyll/markdown files, preserving the original URLs

It's only good enough to convert the pretty restricted HTML that my posts were using. I'm sure if you throw some arbitrary HTML at it, you'll get some quality vomit out of it.


## Dealing with images

This is one part where I fell a bit short. The right thing to do is to bring all the images into the git repo so it's all self-contained.

But I got lazy, so I ended up continuing to point to their original blogger location, which I'm sure I will sorely regret in 2019 when they suddenly disappear.


## Styling the blog

I have no talent for styling, and no css skills, so the goal was to find a fully ready theme. I started using one from [jekyllthemes.org](http://jekyllthemes.org/). But then I decided I didn't like it, so I ended up just ripping off Phil Haack's [blog](http://haacked.com/), because I figure if it was good enough for him, it must be good enough for me (and he [didn't mind](https://twitter.com/haacked/status/420456185121099776)).

If you look carefully, you might notice some hints of similarities :)
