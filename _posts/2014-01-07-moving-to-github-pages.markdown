---
layout: post
title:  "Moving to GitHub pages"
comments: true
categories: blogging
---

For the past few years, I've had my blog hosted on blogger, and for the most part, I hated it. While Windows Live Writer was helping make the authoring experience bearable, in the end there was no getting away from the fact that **I hate HTML**!

On the other hand, **I love Markdown**, so I knew I had to move to a system that let me just use that directly. But when I asked on Twitter, people threw all kind of interesting options at me, and I had to make a choice. In the end, I went with Jekyll/GitHub pages, so I'll use this post to discuss the thought process.

## Lot's of options

Here are some of the other options I looked at based on people's suggestions.

### Ghost

Using [Ghost](https://ghost.org/) was really tempting. It's new and shiny, it has a clean interface, and it has a very nice browser hosted Markdown editor. Also, it runs great on Azure Web Sites, which is what I work on.

But then I realized something else: **I hate databases**. And **I love files** :)

I just didn't want to deal with a system where my posts ended up somewhere in a database. So that was that.

### Orchard

[Orchard](http://www.orchardproject.net/) also has nice [Markdown support](http://www.davidhayden.me/blog/orchard-1.3-features-markdown-support-for-pages-blog-posts-and-content-authoring), which looked like a potential option.

But for the same reason as Ghost, I didn't want to go down that route.


### Sandra.Snow

Several folks suggested that I look at [Sandra.Snow](https://github.com/Sandra/Sandra.Snow), which is a .NET based system inspired by Jekyll. Being a .NET guy, it was tempting instead of using something based on Ruby/Python.

But this came with a big catch: if I used it with GitHub pages, I would need to locally generate the HTML, and then commit that to my repo. And the very thought of committing generated files to a repository makes me sad.

Another big one is that it would not have allowed me to tweak posts online and have them just go live.


### Site44

[Steve Marx](https://twitter.com/smarx) suggested [site44](http://www.site44.com/), which would let me publish my blog simply by adding files to a dropbox folder. And that's certainly a cool way to publish files with no fuss.

But similarly to Sandra.Snow, I would have had to run Jekyll manually to create HTML files each time I want to publish, and I decided that wasn't for me.


## GitHub pages with Jekyll solved most issues

While not perfect, using GitHub pages with Jekyll provides a workflow that best matched what I was looking for:

1. **No database**: it's just a bunch of files. Yeah!
2. **No HTML**: that's not completely true, as I did install Jekyll locally, and when I run it, I get local HTML files. But I think in most cases when I'll want to author a new post, I'll directly push my new Markdown file and let GitHub do the dirty work.
3. **Built-in history**: it's a git repo. Enough said!
4. **Browser based editing**: Github's editor is rather aweful (e.g. compared to Ghost), but it's good enough to tweak existing posts. I hit save, and in under a minute, it's live on my blog. I can do this from my phone if I need to. This would not be possible with Sandra.Snow or Site44.
5. **Collaborative workflow**: if someone finds a typo is my post, they can just send a pull request. ANd then I can accept it without leaving my browser. This is brilliant, and none of the other 4 solutions above provide this.

Well, it's too early to say that the end to end workflow is working great for me, but hopefully time will prove that it was a wise decision, as I'm not planning another move for a while!
