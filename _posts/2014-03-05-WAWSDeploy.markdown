---
layout: post
title:  "WAWSDeploy - a simple tool to deploy to Azure Web Sites"
comments: true
tags: [azure,websites,waws,msdeploy,webdeploy]
---

My preferred method of deploying to an Azure Web Site is using git, partially because this is a feature I've been involved with from the beginning (it's known as [Kudu](https://github.com/projectkudu/kudu)).

However, in some cases, I just need to deploy a bunch of files from my local machine with minimal fuss, and using git is overkill. For these scenarios WebDeploy (aka msdeploy) really shines.

The problem with WebDeploy is that using it from the command line is exceedingly difficult. First, you have to download the publishing profile, and then you extract a bunch of different things from it. From those things, you can put together this charming command line, which will deploy a local folder to your Azure Web Site:

    msdeploy.exe –verb:sync
        -source:contentPath="c:\FolderToDeploy"
        -dest:
            contentPath='MyAzureSite',
            ComputerName="https://waws-prod-blu-001.publish.azurewebsites.windows.net:443/msdeploy.axd?site=MyAzureSite",
            UserName='$myazuresite',
            Password='fgsghfgskhBigUglyPasswordjfghkjsdhgfkj',
            AuthType='Basic'

## WAWSDeploy to the rescue

To make things easier, I wrote a little tool which makes this as simple as it can be. You still need to download the Publish Profile, but then you simply run:

    WAWSDeploy c:\FolderToDeploy MyAzureSite.PublishSettings

So basically, you tell it where your files are, and where they need to go.

To get the tool, you can either build it yourself from the [sources](https://github.com/davidebbo/WAWSDeploy), or get it from [Chocolatey](https://chocolatey.org/packages/WAWSDeploy).

Random notes:

- it's best used for simple sites that don't need any build steps (so not for ASP.NET MVC)
- it's just a fun little tool I wrote on the side, and not a supported Microsoft thing

Let me know if this is useful, and feel free to send a PR if you find issues or want to improve it.

