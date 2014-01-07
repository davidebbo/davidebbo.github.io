---
layout: post
title:  "The easy way to publish NuGet packages with sources"
comments: true
categories: NuGet
---


The standard way to create NuGet packages today is to:

- Create a nuspec file with all the metadata and package dependencies  
- Lay out the files that to want to include  
- Run 'nuget pack' to create the package  
- Run 'nuget push' to push them to the gallery


See [Phil's post](http://haacked.com/archive/2011/01/12/uploading-packages-to-the-nuget-gallery.aspx) more more details on those steps.

While this is pretty straightforward, it can be made yet easier if we take advantage of the fact that your VS project already contains a lot of information that shouldn't have to be repeated.

Today, we are releasing a new nuget.exe feature of that makes this a lot easier.

### Debugging support via SymbolSource.org

The other really exciting thing we'd like to announce today is that we have partnered with the folks at [http://www.symbolsource.org/](http://www.symbolsource.org/) to offer a really simple way of publishing your sources and PDB's along with your package.

Up until now, there really wasn't a great way for package authors to let their users debug into the package's binaries. The user would have needed to download the sources separately from wherever the project is hosted, making sure that they exactly match the state of the binary. They would also need to locate the matching PDBs. That's pretty hard to get right, and most users would generally not bother.

Now with almost no additional effort, package authors can publish their symbols and sources, and package consumers can debug into them from Visual Studio.

### What the package author needs to do

Let's first play the part of the author of a package that contains an assembly, which itself makes use of a library from another package. Let's say that other package is [Clay](http://nuget.org/List/Packages/Clay) as an example.
Step 1: create a project
Let's start by creating a new Class Library project in VS. I'll call it DavidSymbolSourceTest.
Step 2: set some metadata on it
This is an often forgotten step, but it is important to set some basic metadata on your assembly. As you'll see later, it's particularly important with this workflow. To do this, just open the Properties\AssemblyInfo.cs file and change a few values:

{% highlight c# %}
[assembly: AssemblyTitle("DavidSymbolSourceTest")]
[assembly: AssemblyDescription("David's little test package to demonstrate easy package creation")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("David Ebbo")]
// Stuff omitted
[assembly: AssemblyVersion("0.5.0.0")]
{% endhighlight %}

Here, I set a description for the assembly, and put my name as the 'Company' (which is basically the author). I also changed the assembly version to 0.5.
Step 3: bring in our dependencies using NuGet
Next, let's use NuGet to bring in our Clay dependency. I assume you're familiar with the steps to do this, but in case you're not, start [here](http://nuget.codeplex.com/documentation?title=Getting%20Started).

Note that because Clay itself has dependencies, this ends up bringing in 4 packages:

![image](http://lh4.ggpht.com/_jySMpScpTXc/Taa7SkGDKKI/AAAAAAAAAVw/1Z1Hj0Fnrv8/image_thumb%5B1%5D.png?imgmax=800)
Step 4: let's write some code!
In our library, we'll just write some simple code that uses Clay:

{% highlight c# %}
namespace DavidSymbolSourceTest {
 public class Demo {
     public static dynamic GetPersonObject(string first, string last) {
         dynamic New = new ClaySharp.ClayFactory();

         return New.Person(new {
             FirstName = first,
             LastName = last
         });
     }
 }
}
{% endhighlight %}

It just has a little test method with builds a Clay object based on two fields. Pretty boring stuff, but enough to demonstrate the concepts.
Step 5: save your NuGet.org access key
From here on, we'll be using the NuGet.exe command line tool. Make sure you get the latest from [here](http://nuget.codeplex.com/releases/view/58939), or if you already have an older build, run 'nuget update' to self-update it.

Now go to [http://nuget.org/Contribute/MyAccount](http://nuget.org/Contribute/MyAccount) to get your access key, and use nuget.exe save it so you don't have to deal with it every time (so this is a one-time step, not for every project!). e.g.

```
D:\>nuget setapikey 5a50d497-522a-4436-bf90-b65362e65f52
The API Key '5a50d497-522a-4436-bf90-b65362e65f52' was saved for the NuGet
gallery feed (http://go.microsoft.com/fwlink/?LinkID=207106) and the symbol
server (http://nuget.gw.symbolsource.org/Public/NuGet).

```

Note: no, this is not actually my key, but thanks for asking! :)
Step 6: specify additional metadata using a nuspec file
In step 2, we added some metadata in AssemblyInfo.cs, which NuGet can directly understand. Unfortunately, some of the NuGet concepts don't have a matching CLR attribute yet, so we still need a nuspec file to specify the rest.

To create one, just run 'nuget spec' from the folder where the csproj is.

```
D:\DavidSymbolSourceTest\DavidSymbolSourceTest>nuget spec
Created 'DavidSymbolSourceTest.nuspec' successfully.

```

NuGet.exe detects that the nuspec file is meant as a 'companion' to a VS project, and will generate a file with replacement tokens. e.g.

```
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
<metadata>
  <id>$id$</id>
  <version>$version$</version>
  <authors>$author$</authors>
  <owners>$author$</owners>
  <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
  <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
  <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
  <requireLicenseAcceptance>false</requireLicenseAcceptance>
  <description>$description$</description>
  <tags>Tag1 Tag2</tags>
</metadata>
</package>

```
Note how a number of the fields use a token syntax like $version$. This basically means: I don't want to repeat what's already in my AssemblyInfo.cs, so just get values from there.

Now all you need to do is:

- Fill in the fields you care about, like <projectUrl>
- Remove the ones you don't care about. e.g. <iconUrl> if you don't have an icon.


Note that technically, this whole step is optional, and you can omit the nuspec file entirely if you don't need any metadata other than what's in AssemblyInfo.cs. However, since all packages are supposed to specify a <projectUrl> and a <licenseUrl>, in practice it's not a step you'll want to skip.
Step 7: create the package
This is where the new and exciting stuff really starts. Go to the folder where the csproj file is and run:

```
D:\DavidSymbolSourceTest\DavidSymbolSourceTest>nuget pack -sym DavidSymbolSourceTest.csproj
Attempting to build package from 'DavidSymbolSourceTest.csproj'.
Building project for target framework '.NETFramework,Version=v4.0'.
Packing files from 'D:\DavidSymbolSourceTest\DavidSymbolSourceTest\bin\Release'.
Found packages.config. Using packages listed as dependencies
Successfully created package 'D:\DavidSymbolSourceTest\DavidSymbolSourceTest\DavidSymbolSourceTest.0.5.nupkg'.

Attempting to build symbols package for 'DavidSymbolSourceTest.csproj'.
Building project for target framework '.NETFramework,Version=v4.0'.
Packing files from 'D:\DavidSymbolSourceTest\DavidSymbolSourceTest\bin\Release'.
Found packages.config. Using packages listed as dependencies
Successfully created package 'D:\DavidSymbolSourceTest\DavidSymbolSourceTest\DavidSymbolSourceTest.0.5.symbols.nupkg'.

```

Note that we are passing the -sym flag to the 'nuget pack' command, and that we're giving it as input the csproj file!

The command will build the project if needed, and then create both a regular package (DavidSymbolSourceTest.0.5.nupkg) and a 'symbolsource' package (DavidSymbolSourceTest.0.5.symbols.nupkg).

Note how it used the version we had specified in AssemblyInfo.cs in step 2. Likewise, the Author and Description in the package came from there. This happens because of the token replacement logic from step 6.

In addition to the metadata inherited from AssemblyInfo.cs, the package will contain the metadata you explicitly added to the nuspec file, like the Project Url.

And one more thing: it also found our dependency on Clay and added that in the package, again without having to add that explicitly to the nuspec file!
Step 8: push the packages to NuGet.org and SymbolSource.org
Now that we created the packaged, we just need to push them out: one goes to NuGet.org and the other one the SymbolSource.org. This can all be done in one command:

```
D:\DavidSymbolSourceTest\DavidSymbolSourceTest>nuget push DavidSymbolSourceTest.0.5.nupkg
Pushing DavidSymbolSourceTest 0.5 to the NuGet gallery feed (http://go.microsoft.com/fwlink/?LinkID=207106)...
Publishing DavidSymbolSourceTest 0.5 to the NuGet gallery feed (http://go.microsoft.com/fwlink/?LinkID=207106)...
Your package was published.

Pushing DavidSymbolSourceTest 0.5 to the symbol server (http://nuget.gw.symbolsource.org/Public/NuGet)...
Publishing DavidSymbolSourceTest 0.5 to the symbol server (http://nuget.gw.symbolsource.org/Public/NuGet)...
Your package was published.


```
Note that we ran 'nuget push' on the main package, and it automatically pushed the symbol package at the same time.And now we're done, our package is live and ready to be installed from NuGet and debugged with full sources!


### What the package Consumer needs to do

Now let's play the part of the package Consumer that uses this package. Here I'll demonstrate using a simple Console app, though the steps apply equally well to other apps.

**Important note**: these steps are more interesting when done on a different machine than the 'Package Author' steps! If you do them on the same machine, rename or delete the Author project to make sure VS doesn't take any shortcuts on you when debugging (which it will!).
Step 1: set up the VS debugger settings
This is a one time setup step. In VS, go under Debug / Options and Settings, and make a few changes:

- Under General, turn **off** “Enable Just My Code”

- Under General, turn **on** “Enable source server support”. You may have to Ok a security warning.

- Under Symbols, add “http://srv.symbolsource.org/pdb/Public” t the list. Dialog will look like this:


![image](http://lh3.ggpht.com/_jySMpScpTXc/Taa7TdcTz7I/AAAAAAAAAV4/kZYcxrs7zeY/image_thumb%5B4%5D.png?imgmax=800)


Step 2: create a test console app
Make sure you set its Target Framework to the Server profile (see my previous [post](http://blog.davidebbo.com/2011/02/creating-console-apps-that-use-server.html)).
Step 3: use NuGet to bring in our test package
Here is what you'll see in the Online tab of the NuGet dialog:

![image](http://lh3.ggpht.com/_jySMpScpTXc/Taa7UfAnx_I/AAAAAAAAAWA/wF-6qpF7PsE/image_thumb%5B9%5D.png?imgmax=800)

Notice that not only our new package shows up on the public feed, but all the metadata and package dependencies are there as well!

Now click Install to install the package and its dependencies.
Step 4: write some test code to use the package
We'll just call the method we defined and display some output:

```
using System;

namespace ConsoleApplication12 {
 class Program {
     static void Main(string[] args) {
         var person = DavidSymbolSourceTest.Demo.GetPersonObject("David", "Ebbo");
         Console.WriteLine("{0} {1}", person.FirstName, person.LastName);
     }
 }
}


```
Step 5: debug into the package!
This is the final step that makes it all worth it! Set a breakpoint on the line that calls our GetPersonObject method and press F5 to start debugging.

When you hit the breakpoint, click F11 and be amazed!

![image](http://lh5.ggpht.com/_jySMpScpTXc/Taa7U2KXEYI/AAAAAAAAAWI/TKhIIEDnUYQ/image_thumb%5B11%5D.png?imgmax=800)

Here we are debugging into our new package. Here both the sources and PDB files are coming straight from SymbolSource.org!

### Registering with SymbolSource.org

Note that in all the steps below, we never actually went to the [http://www.symbolsource.org/](http://www.symbolsource.org/) web site. The nice thing is that everything can work without even setting up an account on there. Note that SymbolSource does verify that you own the package by checking with nuget.org using your key.

But even though the registration step is optional, it is recommended that you register with the site in order to be able to manage the symbol packages that you upload there. To register, just go to [http://www.symbolsource.org/](http://www.symbolsource.org/) and follow the instructions. During registration, you'll be asked for your NuGet key, which is how your account will get associated with your submissions.

