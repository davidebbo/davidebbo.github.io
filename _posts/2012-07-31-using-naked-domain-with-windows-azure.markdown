---
layout: post
title:  "Using a naked domain with a Windows Azure Web Site"
comments: true
categories: 
---


****

**Update (9/17/2012): as of today, Azure Web Sites have direct support for A record, so the hack below is no longer necessary!**

****

**Warning**: the following is just something that worked for me, and that others asked me about. It is not a Microsoft approved technique, and it could cause your cat to self-combust with no warning. I'm just a guy trying random things here.



Windows Azure Web Sites (WAWS) support custom domain names, as documented on [this page](https://www.windowsazure.com/en-us/develop/net/common-tasks/custom-dns-web-site/). This makes it easy to use a name like [www.davidebbo.com](http://www.davidebbo.com), by setting up a CNAME record in your DNS.

But a lot of people are asking how to make the name just [davidebbo.com](http://davidebbo.com), which is known as a naked domain (aka a bare or root domain). Normally, this is done by setting up an A record, but this requires a stable IP address, which is not currently available in WAWS.

But it turns out that you can use use a CNAME for the naked domain, even though many people say it's a bad idea (more on this below).

I'm not sure if this works with all registrars, but I use NameCheap, and it works with them. Here is what it looks like in the NameCheap DNS records:

![image](http://lh4.ggpht.com/-wAVY5LJ2UTM/UBizU4Xgh4I/AAAAAAAADnc/PUmLE1_kCK8/image_thumb%25255B10%25255D.png?imgmax=800)

So I'm doing two things here:

- First I'm making [http://davidebbo.com](http://davidebbo.com) go to my WAWS  
- Then I'm making [http://www.davidebbo.com](http://www.davidebbo.com) redirect to [http://davidebbo.com/](http://davidebbo.com/). This is optional.



Then I have the following in the Configure tab of my WAWS:

![image](http://lh6.ggpht.com/-326gitmN5Po/UBizAcswG4I/AAAAAAAADnA/jd2qpC-CPRM/image_thumb%25255B7%25255D.png?imgmax=800)

Though really, I only need the last entry since I'm redirecting www to the naked domain. I just left the www entry in there because it doesn't hurt. The first one could go too.



### So what's wrong with doing this?

If you search around, you'll find a number of pages telling you that it's unsupported, and breaks RFC1034 (e.g. see [this page](http://superuser.com/questions/264913/cant-set-example-com-as-a-cname-record)). And I'm sure that the experts will crucify me and call me an idiot for blogging this, but heck, I can live with that!

Personally, I don't care so much about breaking an RFC, as much as I care about breaking my award winning [http://davidebbo.com/](http://davidebbo.com/) web site, which brings me most of my income.

So what might break? From what I'm told, doing this breaks MX records, which matters if you're running an email server under your host name. So if I wanted to be me@davidebbo.com, I probably couldn't. But I don't, so I don't care. It might also affect other types of records that I'm not using.

All I can say is that so far, I'm yet to find something broken about it, and I've heard from several others that they've been using this successfully for a while (not with WAWS, but that shouldn't matter).

Anyway, I think you get my point: try at your own risk! And sorry about your cat.

