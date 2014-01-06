---
layout: post
title:  "Managing database connections in Azure Web Sites"
categories: git azure kudu
---

*This topic is not specific to git publishing, but it's particularly useful in that scenario*

In its first release, the Azure portal had a couple of limitations relating to database connections:

- The database name had to match the connection string, which was quirky
- You could not add external connection strings

The good news is that both limitations have now been removed, as you'll see here.


### Using a SQL Azure database associated with the site

Let's say you'd like to publish the awesome NuGet Gallery to an Azure Web Site. When you create the site in Azure, choose the 'Create with database' option:

![New site](http://lh6.ggpht.com/-XT3zcbwAv1M/UE-4W5vjrFI/AAAAAAAADpM/p3HeYukyzxo/image%25255B37%25255D.png?imgmax=800)

You will see a dialog that looks like this:

![Create site](http://lh6.ggpht.com/-qiiPv_Zl8ds/UE-4XqaAOZI/AAAAAAAADpQ/lBDDRp-3MH8/image%25255B38%25255D.png?imgmax=800)

Let's assume that you don't have a database yet, and you'd like one to be created for you. All you have to do here is give Azure your connection string name (highlighted above).

So where does this 'NuGetGallery' string come from? It's simply the name of the connection string from the app's web.config:

[web.config](![Create site](http://lh5.ggpht.com/-0RHPX8Q8dhc/UE-4YsBn35I/AAAAAAAADpU/cCPW1e2EHd0/image%25255B39%25255D.png?imgmax=800))

This way, you don't need to change your sources to point to the SQL Azure database. You instead rely on Azure to use the right connection string at runtime.

After the following step, you can complete the wizard by either creating a new DB Server or use an existing one. Note that the database itself can be named anything you like (or keep the random name), since it is now decoupled from the connection string name.

At this point, you can just 'git push azure master' the NuGet Gallery sources, and your site is up and running with no further configuration!

Now if you go into the Configure tab for your site, you'll see your associated connection string:

![connection strings](http://lh3.ggpht.com/--d1mgOEUlhE/UE-4ZTMi7nI/AAAAAAAADpY/9n8emYSj_8g/image%25255B40%25255D.png?imgmax=800)

Note that it's hidden by default, but you can choose to display it if you need it (e.g. if you want to connect via SQL management studio). You can even edit it if you want to tweak it!

### Working with external connection strings

In the scenario above, we were using a database that Azure created for us along with the site. In some cases, you will instead need to work with an existing database, which may or may not be hosted on Azure.

In that scenario, you'd create your site without any databases. Instead, you can manually add the connection string in the Configure tab, e.g.

![connection strings](http://lh4.ggpht.com/-1B4_tQYkNh0/UE-4amdKCYI/AAAAAAAADpc/AgxQ7DwmsmI/image%25255B41%25255D.png?imgmax=800)

**Note**: don't forget to click the Save button at the bottom of the page when you're done!

Note that as before, we're naming the connection string after the one in web.config. The only difference is that the value now comes from you instead of coming from Azure.