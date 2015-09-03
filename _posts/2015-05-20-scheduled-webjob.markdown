---
layout: post
title:  "Hooking up a scheduler job to a WebJob"
comments: true
tags: [Azure,WebApps,WebJobs,Scheduler]
---

**Update 9/3/2015: there is now a simpler way to associate a schedule with a WebJob. Please see [this post](http://blog.amitapple.com/post/2015/06/scheduling-azure-webjobs) for details.**

Creating an Azure WebJob that runs on a schedule is a topic that can be a bit confusing. The most important thing to understand about it is that it involves two very separate components:

1. You have a **triggered WebJob** published to a Web App. This WebJob exposes a private endpoint that allows is to get triggered. The WebJob knows nothing about who is doing the firing, and whether a scheduler is involved.
2. Separately, you have an **Azure Scheduler job** that is set up to trigger the WebJob on some schedule.

Those two things end up working together, but they don't really know anything about each other. The only link between them is that the Scheduler Job happens to be pointing at the WebJob's endpoint.

While there are some workflows in the current portal ([https://manage.windowsazure.com/](https://manage.windowsazure.com/)) and in Visual Studio that can make the hookup easier, they don't cover all scenarios. Also, it is good to understand how things work under the cover, and this post explains it by showing the 'manual' steps.


## Assumption: you already have a triggered WebJob

The assumption in this post is that you already know how to publish a WebJob to an Azure WebApp (without any schedule). This can be done in a number of ways (e.g. zip file upload in the portal, WebDeploy from VS, git deployments), and I will not cover those topics here.

This post starts at the point where you have deployed a triggered WebJob, and you're looking to attach a scheduler job to it so it gets triggered on some schedule.

If you look in the WebJobs tab for your host Web App, you'll see something like this:

![image](https://cloud.githubusercontent.com/assets/556238/7735555/df58958c-fef2-11e4-909e-e21588767211.png)


## Getting the publish profile for the Web App that hosts the job

To download it, just Go to the dashboard of your WebApp in the [portal](https://manage.windowsazure.com/), and click 'Download the publish profile'.

![image](https://cloud.githubusercontent.com/assets/556238/7733585/f00ef654-fee4-11e4-9909-acb2b8c8882d.png)

You can do the same thing in the Preview Portal (https://portal.azure.com/), which can be necessary if using an API App (which doesn't show up in the regular portal):

![image](https://cloud.githubusercontent.com/assets/556238/7735267/021964d6-fef1-11e4-83d8-fa9a8f16d742.png)

In that file, you will find the credentials (they are unique per site), which you will use later in this post. They'll look like this (no these are not real!):

    userName="$DavidWebJobTestApp"
    userPWD="WauLFsqoLggpXfjT2z9Hq27kXS6luqn6F3ncRa9bjQjPssCBqB5jKu2Zk4Ed"


## Creating a scheduler job

Find the scheduler node in the left bar in the portal:

![image](https://cloud.githubusercontent.com/assets/556238/7733687/a1fd4ff0-fee5-11e4-9ef1-3b693ded26a5.png)

If you don't have any scheduler jobs, it'll look like this:

![image](https://cloud.githubusercontent.com/assets/556238/7733713/ceeca448-fee5-11e4-808e-375f5f63a547.png)

Click the link to create one, which opens the bottom pane under App Services / Scheduler / Custom Create.

A scheduler job belongs to a **Job Collection**. So if you don't yet have a collection, you'll need to create one. Or if you already have one, you can add a new job to it. Here we'll create a new one, in the same region as the site (not required, but preferable):

![image](https://cloud.githubusercontent.com/assets/556238/7733794/67ce580a-fee6-11e4-80af-420314104e39.png)

Now, there is a somewhat complex step, as you need to assemble the trigger URI to your WebJob. It looks like this:

    https://{userName}:{password}@{WebAppName}.scm.azurewebsites.net/api/triggeredwebjobs/{WebJobName}/run

Let's look at the tokens you need to replace:

- The username and password are from the Publish Profile above
- WebAppName is the WebApp that hosts your WebJob. e.g. *davidwebjobtestapp*
- WebJobName is the name you gave to your WebJob (not to be confused with the name you give to your **scheduler job**!). e.g. *triggeredwebjobs*

Putting it all together, you get a full URL that looks like this:

    https://$DavidWebJobTestApp:WauLFsqoLggpXfjT2z9Hq27kXS6luqn6F3ncRa9bjQjPssCBqB5jKu2Zk4Ed@davidwebjobtestapp.scm.azurewebsites.net/api/triggeredwebjobs/MyWebJob/run

You can now fill in the next step of the wizard:

- Give some name to your scheduler job. It doesn't have to match the WebJob's name, but it might be a good idea so you remember what it points to (here I gave them different names).
- Action type should be https.
- Method should be POST.
- URI is what we cooked up above.
- Leave the body blank.

![image](https://cloud.githubusercontent.com/assets/556238/7735325/7385ee3c-fef1-11e4-9caa-fb8c1b5bbc2a.png)

On the next step, we'll set up the schedule. e.g. let's make it fire every hour for the next month:

![image](https://cloud.githubusercontent.com/assets/556238/7735501/761acf36-fef2-11e4-9c00-35b3f19f72e1.png)

And that's it, the scheduler job is set up!

**Note**: by default, the job collection is created in Standard mode. But if you like you can switch it to Free mode. There are a number of limitations in Free mode, but it is good enough if you are just learning about the feature.

To switch it to Free, go to the Scale tab for the Job Collection:

![image](https://cloud.githubusercontent.com/assets/556238/7735605/22a7fc9c-fef3-11e4-86c1-3ee3500e1aab.png)

## Monitoring your WebJobs

You can do two types of monitoring:

- Scheduler level: you can ask the scheduler for a history of what it has fired. This is the 'client view'.
- WebJobs level: you can ask the WebJobs for what it has received. This is the 'server view'.

For the scheduler view, just go to the history tab in your scheduler job collection:

![image](https://cloud.githubusercontent.com/assets/556238/7735871/de4bdd64-fef4-11e4-88cb-b75cea921c55.png)

For the WebJobs view, click on the Logs link, which you can see in the first image in this blog post. This takes you to the WebJobs dashboard, e.g.

![image](https://cloud.githubusercontent.com/assets/556238/7735789/4e47a946-fef4-11e4-8b30-eccc1b70d955.png)

If you feel like your WebJobs are not running, you may need to check both places. For instance, if the password you entered is incorrect, the scheduler would tell you it got an authentication error, while the WebJobs dashboard won't have received anything at all.

## Cleaning up WebJobs and Scheduler Jobs

Note that if you delete a WebJob (or the Web App that hosts it), the scheduler job pointing to it will keep firing, and getting back errors since there is no one listening. So you probably want to delete the scheduler jobs as well.

Or if you don't have any scheduler jobs left, just delete the whole job collection, to make sure that you don't incur charges related to it.