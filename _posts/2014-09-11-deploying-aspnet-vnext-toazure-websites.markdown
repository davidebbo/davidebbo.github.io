---
layout: post
title:  "Deploying ASP.NET vNext apps to Azure Websites"
comments: true
tags: [Azure,Websites,aspnetvnext]
---

We just added some preliminary support for deploying [ASP.NET vNext](http://www.asp.net/vnext) projects to Azure Websites using git.

You can easily try it yourself: Clone my [test project](https://github.com/davidebbo-test/AspNetVNextWebApp). Then Create an Azure Website [with git enabled](http://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/). Then just push the repo to Azure.

And that's it, your ASP.NET vNext site is up an running in Azure!

Here are various additional notes:

- the support is alpha-level at this point, just like everything relating to ASP.NET vNext
- it only support vNext solutions created by Visual Studio 2014 CTP
- we've only tested with the alpha3 release of the K runtime
- the first deployment takes a bit over a minute as it has to download a bunch of NuGet packages. Subsequent deployments are quite a bit faster
- when running the deployed site, the cold start time is on the slow side. vNext is still very new, and there are lots of things left to tune and optimize!

Anyway, give it a try, and let us know what you think! Please report issues to [https://github.com/projectkudu/kudu](https://github.com/projectkudu/kudu).
