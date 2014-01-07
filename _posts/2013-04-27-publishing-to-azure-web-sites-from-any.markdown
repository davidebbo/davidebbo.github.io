---
layout: post
title:  "Publishing to Azure Web Sites from any git/hg repo"
comments: true
categories: 
---


Windows Azure Web Sites provide a nice selection of ways to deploy a site from source code. You can deploy from:

- a local git repository
- a TFS project
- a git project on GitHub
- a git or Mercurial project on Bitbucket
- a git or Mercurial project on CodePlex
- code in a Dropbox folder


One thing that all these approaches have in common is that you own the code. e.g. in the GitHub case, you must be an admin on the project. The reason is that Azure needs to set up a hook in the project to enable continuous deployment, and only project owners can do that.



### Deploying 'external' repositories you don't own

In some scenarios, it can be interesting to deploy a site based on sources that you don't own. For example, you might want to deploy your own instance of the NuGet gallery, but you have no intention to modify the source. You're happy with it as is, and you just want to deploy it.

To cater to this scenario, we added a new 'External repository' entry in the Azure portal:

![image](http://lh6.ggpht.com/-dGs0OZM0cQI/UXywfwpKscI/AAAAAAAAD2Q/KVgW9FptfPg/image_thumb%25255B2%25255D.png?imgmax=800)

**Note**: the 'External repository' entry is using the git icon, which is technically incorrect since it supports both git and Mercurial. We just didn't have time to come up with a better icon for it for this initial release! We'll probably change that later.

Once you pick that, the next page in the wizard is pretty simple: you just paste any http(s) git or Mercurial URL and you're good to go!

![image](http://lh4.ggpht.com/-G8ml__1f2KQ/UXywgqZZj8I/AAAAAAAAD2g/JNAFpX17lVM/image_thumb%25255B5%25255D.png?imgmax=800)

And as soon as you Ok the dialog, a deployment from that repository gets triggered.



### What about updates?

One important point about this mode is that it doesn't support continuous deployment. This is because Azure cannot possibly register for change notifications on an arbitrary repo that you don't own.

Concretely, that means that your site will not be automatically deployed when the repo is updated. Instead, you need to tell it when you want to pick up changes, by clicking the Sync button in the Deployments page:

![image](http://lh4.ggpht.com/-b7HwYPZQtcs/UXywhZwen3I/AAAAAAAAD2s/wlzQbwI6EKc/image_thumb%25255B8%25255D.png?imgmax=800)

While this feature may not see the same kind of usage as the full continuous deployment workflows, it has its uses and nicely completes the overall source deployment story in Azure Web Sites.

