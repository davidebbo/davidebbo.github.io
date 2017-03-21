---
layout: post
title:  "Using issues to host GitHub Pages screenshots"
comments: true
tags: [Blogging]
---

[GitHub pages](https://pages.github.com/) make for a nice blogging platform if you like working with git and markdown. I [blogged](http://blog.davidebbo.com/2014/01/moving-to-github-pages) about it earlier this year.

But when it comes to embedding screenshots in your posts, it's not obvious what the best approach is. In this post, I'll first discuss a couple options that I didn't like, and then suggest an approach that I think works pretty well.

## Bad option #1: store your images in your git repo

Since your GitHub Pages blog is based of a git repo, there is something to be said for storing your images in the repo itself, so that they are versioned alongside your markdown.

The steps to do this would go something like this:

- Use something like the [Snipping Tool](http://windows.microsoft.com/en-us/windows7/products/features/snipping-tool) to select the desired area of the screen.
- Save it in the images folder of your GitHub Pages repo, giving it a meaningful name
- Reference it from your post using something like this: `![something cool](/images/SomethingCool.JPG)`
- Add and commit the image to your repo


#### Why I'm not using this approach

There are a couple reasons why I don't use this approach.

The first is that it's a heck of a lot of steps that you have to go through for each image. When you've previously used a tool like Windows Live Writer where you just drag and drop images into your post, it's a major drop in productivity.

The second reason is that I have a serious aversion to storing binaries in git repos. Each image forever remains in the repo history, making it irreversibly bloated. Sometimes, you may need to make adjustments to previously published images, piling on more crap onto your repo. You may see it as a benefit to have the full and true history tracked by your repo, but for me the drawbacks really outweigh the benefits.

## Bad option #2: use an external image sharing site

I use MarkdownPad 2 to author my markdown. It has a convenient option to upload an image to Imgur and insert the right markdown. Other markdown editors have similar options.

#### Why I'm not using this approach

I simply don't trust those external image hosting services to stay around forever. Have you heard of Twitpic? It almost disappeared a month a ago along with all your cherished images. It got [saved at the eleventh hour](http://techcrunch.com/2014/10/25/twitpic-data-will-stay-alive-for-now-thanks-to-an-agreement-with-twitter/), but clearly it's quite a gamble to trust these sites for posterity. Don't do it!

## Suggested option: use GitHub issues to store your images

GitHub issues has a very cool feature that lets you effortlessly [drop images into an issue](https://help.github.com/articles/issue-attachments/). It automatically uploads it to its cloud and inserts the right markdown.

Here is how we can abuse this feature for our GitHub Pages posts:

- For each blog post, create a new issue, to keep things organized. e.g here is the [issue](https://github.com/davidebbo/davidebbo.github.io/issues/19) I'm using for this post.
- When you need to embed a screenshot, get it into your clipboard using your tool of choice. I just use the Windows [Snipping Tool](http://windows.microsoft.com/en-us/windows7/products/features/snipping-tool).
- Go to the GitHub issue and paste it right into the comment. And bang, magic happens and GitHub gives you the markdown pointing you the cloud hosted image. It will look like `![image](https://cloud.githubusercontent.com/assets/556238/5241760/e17ae9f4-78d9-11e4-86f5-6e168adcea87.png)`
- Paste that markdown in your post, and you're done!

Here is an example:

![image](https://cloud.githubusercontent.com/assets/556238/5241760/e17ae9f4-78d9-11e4-86f5-6e168adcea87.png)

Various tips about using these 'fake' GitHub issues:

- If you have multiple images, you can add then to the same comment in the GitHub issue. Or you could create additional comments in the same issue if you prefer. It really doesn't make much difference.
- It's a good idea to name the issue after the blog post, to keep things tidy
- When I create these issues, I close them right away, since they are not real issues.

#### Why I like this approach

This approach is undeniably a hack, as it uses GitHub issues in ways they were not intended. Despite that, it adds up to a decent solution for me.

First of all, the workflow is quite efficient. Once you have the issue created, you can go through all the steps in just a few seconds (after having captured the screenshot in the Snipping tool). It's still not quite the Live Writer experience, but it gets pretty close.

And second of all, it keeps your repo mean and lean!

The elephant in the room here is that this technique is not fundamentally different from my *Bad option #2* above. In both cases, the images are stored in some cloud somewhere, and if that cloud was to evaporate, your images would be gone.

So what makes one *bad* and the other one *good*? Well, the reasoning is that by using GitHub pages, you're already trusting GitHub to host your repo as well as your live blog. So in a sense, by relying on Issues (on that same repo), you're just extending your use of GitHub, rather than take an additional external dependency. And I generally trust GitHub to stay around longer than those various image hosting services that lack any tangible business model.

Now, if you're reading this post and see a broken image above, you can laugh your ass off and say that I was wrong.
