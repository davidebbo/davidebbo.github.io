---
layout: post
title:  "ARMClient: a command line tool for the Azure API"
comments: true
tags: [Azure,API,Websites]
---

ARMClient is a console application that makes it easy to send HTTP requests to the new Azure Resource Manager REST API. Note that it only supports the new Azure API (ARM) and not the older one (RDFE).

## A few notes before we start

At this point, ARMClient is not an official Microsoft tool. It is an OSS Project written primarily by [suwatch](https://github.com/suwatch). You can find it on [https://github.com/projectkudu/ARMClient](https://github.com/projectkudu/ARMClient). We are releasing it because we think it can be useful to others. Based on the feedback, we'll see what direction we will take with it.

Also, note that this post is primarily about the ARMClient tool, and is not meant to be a general tutorial for the ARM API. You can check out the [REST API Reference](http://msdn.microsoft.com/en-us/library/azure/dn790568.aspx) to learn about some of the concepts. You can also find lots of examples on the [ARMClient wiki](https://github.com/projectkudu/ARMClient/wiki).

If you get stuck figuring out how to do something with ARMClient, feel free to discuss in an [ARMClient GitHub issue](https://github.com/projectkudu/ARMClient/issues/new).


## Why this tool

Today, there are two primary ways of automating the Azure API from the command line:

- [Azure PowerShell](http://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/): is used on Windows, and is the great fit for PowerShell users (to state the obvious!)
- [Azure Cross-Platform Command-Line Interface](http://azure.microsoft.com/en-us/documentation/articles/xplat-cli/) (aka xplat-cli): this is written in Node, and runs on all platforms.

Both of these options offer a fairly high level of abstraction over the Azure API. e.g. to create a site with xplat-cli, you would run something like `azure site create mywebsite`.

By contrast, ARMClient makes no effort to abstract anything, and instead lets you use the raw API directly. The closest thing you should compare it to is good old cURL. And while you *could* use plain cURL to do the same, ARMClient makes it a lot easier, both because it helps with authentication and because its syntax is simpler/cleaner.

There are pros and cons to each approach. One big benefit of the ARMClient approach is that you can call any supported ARM API. With PowerShell/xplat-cli, there can be delays before new APIs get abstracted into new commands (of course, ideally that wouldn't be the case, but as things stand, it does happen).

On the downside, some will find that the ARMClient approach is too low level, and that they don't want to work at the raw HTTP/JSON level. Though I will say that it is not as scary as it may sound at first :)

## Getting ARMClient

ARMClient is distributed via [Chocolatey](https://chocolatey.org/). After installing Chocolatey (if you don't already have it, you've been missing out!), just run:

    choco install armclient

And you'll magically have ARMClient.exe on your path. Run it without parameters to get the help text.

## Authenticating with Azure

There are two main ways you can do this.

The first is by logging in interactively using your Microsoft Account (or your Work/School account). You don't need to run any special commands to do this. Instead, the first time you make a regular ARMClient request, it will pop up a browser window and prompt you for credentials. This is probably where you want to start to play around with this tool. 

The second is to use a Service Principal. My [earlier post](http://blog.davidebbo.com/2014/12/azure-service-principal.html) explains what it is, and how to create one. This is what you would use in automated scenarios, like in a CI server.

To take the example from that post, after setting things up, you end up with something like this (no, they're not valid credentials!):

- Tenant ID: `361fae6d-4e30-4f72-8bc9-3eae70130332`
- AppId/Username: `dc5216de-6fac-451a-bec4-9d2fb5568031`
- Password: `HGgDB56VAww1kct2tQwRjOWBSkUOJ3iMDGEiEjpBZEQ=`

You use these three pieces to authenticate as follows ('spn' stands for Service Principal Name):

    armclient spn 361fae6d-4e30-4f72-8bc9-3eae70130332 dc5216de-6fac-451a-bec4-9d2fb5568031 HGgDB56VAww1kct2tQwRjOWBSkUOJ3iMDGEiEjpBZEQ=

Note that whichever authentication method you use, armclient caches the resulting token in your `%USERPROFILE%\.arm` folder (in encrypted form). If you want to clear the cache, you can just run `armclient clearcache`.

## Making requests

Now that we're authenticated, it's time to make requests!

Let's start with something simple and list our subscriptions:

    armclient GET /subscriptions?api-version=2014-04-01

Which returns something like this (you may have multiple):

```json
{
  "value": [
    {
      "id": "/subscriptions/9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8",
      "subscriptionId": "9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8",
      "displayName": "MSDN",
      "state": "Enabled"
    }
  ]
}
```

Note how the API version in passed on the query string. This is true of all calls to the ARM API.

Since most requests are made on a subscription, lets make our life easier and set up a variable for the root of the path that captures the subscription:

    set SUB=/subscriptions/9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8

Now let's list the [resource groups](http://azure.microsoft.com/en-us/documentation/articles/azure-preview-portal-using-resource-groups/) in our subscription:

    armclient GET %SUB%/resourceGroups?api-version=2014-04-01

This will return something like this:

```json
{
  "value": [
    {
      "id": "/subscriptions/9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8/resourceGroups/MyResGroup",
      "name": "MyResGroup",
      "location": "northeurope",
      "properties": {
        "provisioningState": "Succeeded"
      }
    },
	  // various other resource groups
  ]
}
```

Now let's list all Websites in this resource group:

    armclient GET %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites?api-version=2014-11-01

To create a new Website, we'll need to do a PUT. Note that PUT requests are used both for creation and update operations.

The minimal body we need to pass in looks like this:

```json
{
  "location": "North Europe",
  "properties": { }
}
```

Put that in a CreateSite.json file and run:

    armclient PUT %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite%?api-version=2014-11-01 @CreateSite.json

Note how `@CreateSite.json` (with the `@` sign) means it's coming from a file. You could also place the content inline if it's small, e.g.

    armclient PUT %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite?api-version=2014-11-01 "{location: 'North Europe', properties: {}}"

Notice how it returns a response containing the state of the new site object (e.g. its host names and many other things).

Now let's change the site's PHP version to 5.6. We'll use this body:

```json
{
  "properties": {
    "phpVersion": "5.6"
  }
}
```

And then make this request:

    armclient PUT %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite/config/web?api-version=2014-11-01 @RequestBodies\SetPHPVer.json

Note that phpVersion is a site config property, and not a site level property, hence the extra `config/web` at the end of the path.

Now, here is how you would stop and start the site:

    armclient POST %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite/stop?api-version=2014-11-01
    armclient POST %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite/start?api-version=2014-11-01


Finally, let's delete this site:

    armclient DELETE %SUB%/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite?api-version=2014-11-01

## Using ARMClient in PowerShell scripts

Now let's see how we can make use of ARMClient in a PowerShell script. Let's use it to add an App Setting to a site.

One tricky thing about App Settings is that you need to roundtrip the whole collection if you want to add one. This gives us an interesting challenge, as we need to GET them, modify them and then PUT them back.

Here is how we can do it:

```
$sitePath = "/subscriptions/9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8/resourceGroups/MyResGroup/providers/Microsoft.Web/sites/MyCoolSite"
$res = ([string] (armclient POST "$sitePath/config/appsettings/list?api-version=2014-11-01")) | ConvertFrom-Json
$res.properties | Add-Member -Force "foo" "From PowerShell!"
$res | ConvertTo-Json | armclient PUT "$sitePath/config/appsettings?api-version=2014-11-01"
```

So here is what happens:

- We first get the App Settings. Note that this is done via a POST to the `list` verb instead of a GET, because it involves secrets that a plain reader should not see. This is a pattern that you will see in various places in the ARM API.
- We then convert the JSON output into a PowerShell object
- Now we use `Add-Member` to add an App Setting in to the `res.properties` object
- We then convert the PowerShell object back to JSON and pipe it into ARMClient to do the PUT. Note that ARMClient supports getting input from stdin (instead of being on the command line) for this kind of piping scenarios.

## Give us feedback!

Please let us know what you think about this tool. You can post comments here, or open issues on https://github.com/projectkudu/ARMClient. And feel free to send a pull request if you want to get a change in. 