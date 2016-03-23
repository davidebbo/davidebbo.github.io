---
layout: post
title:  "Calling the Azure ARM API using plain REST"
comments: true
tags: [Azure,API,Websites]
---

When it comes to calling the Azure ARM API, the standard approach is to reference the relevant NuGet packages to get the right client API. I have a complete example of doing this [here](https://github.com/davidebbo/AzureWebsitesSamples/tree/master/ManagementLibrarySample). Specifically, that sample is using [Microsoft.Azure.Management.WebSites](https://www.nuget.org/packages/Microsoft.Azure.Management.WebSites).

There is nothing wrong with this approach, which gives you a high level API experience with intellisense.

## But I don't want to use a library!

On the other hand, some developers with knowledge of the API prefer to be close to the wire and call the REST API directly, without using specific libraries. And doing this with the Azure API is actually pretty easy, once you get passed the authentication part.

I posted a full sample [on GitHub](https://github.com/davidebbo/AzureWebsitesSamples/tree/master/HttpClientSample), so you may want to start by looking at that. The sample assumes that you have already set up a [Service Principal](http://blog.davidebbo.com/2014/12/azure-service-principal.html) to access your Azure subscription.

The first step is to obtain an authentication token for your Service Principal. The sample includes a helper function to do this, that you can copy in your code. I won't go into the details of the helper (which is really just a REST call), but calling it is straightforward: 

```
string token = await AuthenticationHelpers.AcquireTokenBySPN(
	tenantId, clientId, clientSecret);
```

Once you have the token, you're ready to create your HttpClient:

```
using (var client = new HttpClient())
{
    client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
    client.BaseAddress = new Uri("https://management.azure.com/");

    // Now you can party with your HttpClient!
}
```

All it's doing is set the Authorization header with the token, and set the base URL so we don't have to specify it on every request.

Now that we have an HttpClient, we're ready to call the Azure ARM API directly. e.g. creating a resource group will look like this:

```
using (var response = await client.PutAsJsonAsync(
    $"/subscriptions/{Subscription}/resourceGroups/MyResourceGroup?api-version=2015-11-01",
    new
    {
        location = Location
    }))
{
    response.EnsureSuccessStatusCode();
}
```

And creating an App Service Plan within that resource group looks like this:

```
using (var response = await client.PutAsJsonAsync(
    $"/subscriptions/{Subscription}/resourceGroups/MyResourceGroup/providers/Microsoft.Web/serverfarms/MyFreePlan?api-version=2015-08-01",
    new
    {
        location = Location,
        Sku = new
        {
            Name = "F1"
        }
    }))
{
    response.EnsureSuccessStatusCode();
}
```

Stopping a Web App is an action, so it requires a POST instead of a PUT:

```
using (var response = await client.PostAsync(
    $"/subscriptions/{Subscription}/resourceGroups/MyResourceGroup/providers/Microsoft.Web/sites/MyApp/stop?api-version=2015-08-01",
    null))
{
    response.EnsureSuccessStatusCode();
}
```

And deleting a Web App is of course a DELETE:

```
using (var response = await client.DeleteAsync(
    $"/subscriptions/{Subscription}/resourceGroups/{ResourceGroup}/providers/Microsoft.Web/sites/{WebApp}?api-version=2015-08-01"))
{
    response.EnsureSuccessStatusCode();
}
```

I have a few more examples in the sample app, so check it out. Note that even though I'm focusing on Azure App Service (where I work), you can use this approach to call *any* ARM APIs: Virtual Machines, Storage, etc... 

## What if you don't know the API?

The simplest way to learn the API in an interactive way is to use Azure Resource Explorer ([https://resources.azure.com/](https://resources.azure.com/)).

Using this, you could have come up with all the samples above without any prior knownledge of the API. In fact, you can even 'practice' making the calls in Resource Explorer before making the REST calls in C#. Since C#'s anonymous object syntax is very JSON like, it's simple to translate from one to the other.

## What about other languages?

That's another benefit of the straight REST approach. Since we're not relying on any libraries, it doesn't really matter what language you use. As long as you know how to make http requests, you're set!
