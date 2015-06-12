---
layout: post
title:  "Automating Azure on your CI server using a Service Principal"
comments: true
tags: [Azure,Websites,API]
---

**Scenario**: you have a CI machine where you need to automate the Azure API. It only needs access to a specific set of resources, and you don't want it to be able to do more than that.

**Solution**: you can create a *Service Principal* account and give it just the set of permissions that it needs. This is a far better solution than using a Management Certificate, which has full power over a subscription.

I'll also give a disclaimer here: I work on the Azure Websites team, and not on the Identity team. I'm not an expert on that topic, but what I describe below is a technique that should work well.

## A tale of two APIs

Azure supports two different REST APIs:

- the old one, known as RDFE
- the new one, known as ARM (Azure Resource Manager)

Many people are still using RDFE, as it's been the only API for years. But ARM is what the cool kids are doing now, and it is worth taking the time to switch over. Besides being a nicer REST API, it supports new key concept like Resource Groups, Deployment Templates and RBAC (Role Based Access Control).

The reason I'm mentioning this is that the technique described here is exclusively for ARM.

## A few notes before we start

I have to warn you that the steps to initially create a Service Principal are fairly complex, kind of ugly, and in some cases rather illogical. I do know that this will get easier in the future, but for now it's a bit of necessary pain to get this going.

The good news is that after those one-time steps, the workflow gets pretty reasonable, and is quite powerful.

Another area to discuss is the two types of Azure accounts:

- Microsoft account (aka Live ID or Passport ID if you're an old-timer)
- Work or School account (aka Org ID)

The steps below can work with both, but since most people today use a Microsoft account, that's what I'm using in the examples.

## Creating an AAD application

The first thing you'll need to do is create an AAD application. To do this, go to the Current  Azure portal (https://manage.windowsazure.com/) and choose Active Directory in the left pane.

![image](https://cloud.githubusercontent.com/assets/556238/5464701/dd2558ee-8540-11e4-97f5-3e4c02070356.png)

As a Microsoft Account user, you should have an active directory named after you:

![image](https://cloud.githubusercontent.com/assets/556238/5464705/f906bba2-8540-11e4-9839-aeec2fcfe677.png)

Click on its name, and then click on APPLICATIONS. It'll probably say that you don't have any. Click to add one:

![image](https://cloud.githubusercontent.com/assets/556238/5464731/33020276-8541-11e4-867b-d3f3fa2982cd.png)

In the next dialog, choose *Add an application that my organization is developing*:

![image](https://cloud.githubusercontent.com/assets/556238/5464744/66e49338-8541-11e4-9ba0-c7d66e8dd25e.png)

Now you'll need to give an answer that probably won't make more sense. Our goal is to automate Azure from our client, yet here you have to tell it to create a Web app. Just go along with it. I warned you some steps would not be too logical!

![image](https://cloud.githubusercontent.com/assets/556238/5464754/9d4323f4-8541-11e4-8ac1-0ef7ed79add9.png)

Now it's asking you for two URLs. In our scenario, using URLs here doesn't really make sense (it does for other scenario). But you'll want to enter some recognizable URL, as we'll need it later during role assignment. e.g. I use http://DavidsAADApp, which is bogus as a URL, but is recognizable to represent my app (this will get cleaner in the future).

![image](https://cloud.githubusercontent.com/assets/556238/5464798/09ef78c2-8542-11e4-87d5-9c50167db765.png)

Congratulations, you now have an AAD application! In there, click on the CONFIGURE tab:

![image](https://cloud.githubusercontent.com/assets/556238/5464836/90a8f0c8-8542-11e4-9a5d-f484b058d6ab.png)

First, find the Client ID and save it. This will basically be your username:

![image](https://cloud.githubusercontent.com/assets/556238/5465681/1dc51758-8551-11e4-8c10-f47a00f34fd7.png)

Now go to the Keys section, click on the drop down, and pick 1 or 2 years:

![image](https://cloud.githubusercontent.com/assets/556238/5464898/ba075d50-8543-11e4-9d39-b65e98d88d50.png)

After you hit save at the bottom, it will display your key, which is basically your Service Principal account password. Save it and store it in a secure place (like a password manager). You will never see it again in the portal!

![image](https://cloud.githubusercontent.com/assets/556238/5465031/911c4818-8545-11e4-88d4-bbd8fbf6d56d.png)

One last thing you need to do is get your tenant ID. The way to do this is a bit harder than it should be (I know, I know...). Click on the View Endpoints button in the bottom bar:

![image](https://cloud.githubusercontent.com/assets/556238/5465439/b03f9cde-854c-11e4-827e-955df4188757.png)

It will show you a bazillion URLs. Click copy on the first one (any of them will do). I will look like this:

    https://login.windows.net/361fae6d-4e30-4f72-8bc9-3eae70130332/federationmetadata/2007-06/federationmetadata.xml

The GUID in there is your tenant ID, which you'll need later. 

It was complex to get here but the summary is that you now have a Service Principal account with a username and a password. And we also have our tenant ID:

- Username: dc5216de-6fac-451a-bec4-9d2fb5568031
- Password: HGgDB56VAww1kct2tQwRjOWBSkUOJ3iMDGEiEjpBZEQ=
- Tenant ID: 361fae6d-4e30-4f72-8bc9-3eae70130332

Now let's move on...

## Assigning roles to your Service Principal

You have a Service Principal account, but right now it's not allowed to do anything. You'll need to use [Azure PowerShell](http://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/) to do this (until the [Preview Portal](https://portal.azure.com/) adds support for it).

Here, you'll want to log in as your Microsoft identity in order to grant roles to your Service Principal identity (conceptually: you're the boss, and you set permissions for your 'employee').

```
Switch-AzureMode -Name AzureResourceManager
Add-AzureAccount # This will pop up a login dialog
```

Now, you can assign roles to your Service Principal. e.g. let's give it access to one of the resource groups in our subscription. You can use either App ID Uri or Client ID as the value for the  `-ServicePrincipalName` parameter.

    New-AzureRoleAssignment -ServicePrincipalName http://DavidsAADApp -RoleDefinitionName Contributor -Scope /subscriptions/9033bcf4-c3c2-4f82-9e98-1cc531f1a8a8/resourceGroups/MyResGroup

Or if you want it to have access to the whole subscription, just leave out the Scope:

    Select-AzureSubscription -SubscriptionId <subscription-id>
    New-AzureRoleAssignment -ServicePrincipalName http://DavidsAADApp -RoleDefinitionName Contributor

If you run `Get-AzureRoleAssignment`, you should see the assignment.

## Using your Service Principal account

So we've finally come to the point where you can make use of this!

We're going to use PowerShell again, but this time not as ourselves, but as the Service Principal identity. e.g. this is what you would do on your CI server, where you'd never want it to use your own identity.

To make sure that we're not cheating, let's start by removing all identities that PowerShell knows about. You can list them using `Get-AzureAccount`, and then run `Remove-AzureAccount YourLiveID@live.com` to remove it.

Now, let's get our Service Principal creds into a `PSCredential` object, as described in [this post](http://blogs.msdn.com/b/koteshb/archive/2010/02/13/powershell-creating-a-pscredential-object.aspx):

```
$secpasswd = ConvertTo-SecureString "HGgDB56VAww1kct2tQwRjOWBSkUOJ3iMDGEiEjpBZEQ=" -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ("dc5216de-6fac-451a-bec4-9d2fb5568031", $secpasswd)
```

**Security note**: because you need to use the key explicitly in this command, you'll want to avoid having it as is in your script (or it might end up getting pushed to a public repo by mistake!). Instead, you'd set up the CI server to make it available to you as an environment variable, and use that instead (or something along those lines).

We are now able to add the Service Principal account, e.g.

    Add-AzureAccount -ServicePrincipal -Tenant 361fae6d-4e30-4f72-8bc9-3eae70130332 -Credential $mycreds

PowerShell is now using your Service Principal identity, and finally, we're able to do stuff! Let's list all the resources (e.g. Websites, databases, ...) in the resource group that we were granted access to:

    Get-AzureResource -ResourceGroupName MyResGroup

This should work!

But if we try it on some other resource group that we were not given access to, it will fail. e.g.

    Get-AzureResource -ResourceGroupName OtherResGroup

This is RBAC doing its magic.

## Conclusion

I know, it feels like a lot of steps to do something simple. Those steps will definitely get easier in the near future (there will be a way to create the Service Principal with one PowerShell command). But for now, with a little extra work it will let you automate your Azure Account in all kind of interesting ways, with the power of RBAC scoping access to exactly what it should be.
