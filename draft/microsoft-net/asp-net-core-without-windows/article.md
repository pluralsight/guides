# Introduction

When .NET Core was announced at Visual Studio Connect() in 2014, one of the most exciting features was that it would be supported across multiple platforms namely Windows, macOS and various distributions of Linux.  However, this also introducted a challenge, which was tooling.  The preferred tool for writing ASP.NET applications at the time was Visual Studio, running on Windows, deploying to Windows.  What would the tooling and developer experience look like in the abscence of Windows?

At the annual Microsoft Build developer conference, six months later in April 2015, Microsoft introduced the world to Visual Studio Code.  Inspired by Visual Studio for Windows, Visual Studio Code is a lightweight text editor which is both free and cross-platform with builds for Windows, macOS and Linux which are conveniently the same platforms targeted by .NET Core.  Built on the Electron shell from Github and written in TypeScript, Visual Studio Code is not even a .NET application, even on Windows.  Eventually, Visual Studio Code was made open source on Github.  

With the addition of support for extension authoring, the capabilities of Visual Studio Code began to increase very quickly.  Today, with the help of extensions, Visual Studio Code is a major player and worthy competitor to other established editors including Visual Studio, Sublime Text and Atom.  This guide will look at what is needed to setup Visual Studio Code, along with the .NET Core SDK, on any of the three supported platforms to create a database-driven web application with a REST-ful API.

# Installation

Let's look at how to set up the development environment.  In addition to the .NET Core SDK and Visual Studio Code, you'll need Docker to run an instance of SQL Server.  I'll be using macOS here but the process will be similar for Windows as well.  Linux will also work but the Docker install will be slightly different.  This guide will focus on macOS and Windows.

### .NET Core

The home page for all things .NET (including .NET Core) is http://dot.net.  

![](https://storage.googleapis.com/ps-dotnetcore/dotnethome.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 1 - The .NET home page**

Clicking on the *Downloads* link will take you to a page where you can select between the .NET Framework, .NET Core and Xamarin.  This guide will use .NET Core so click on that option.  The next page will have the downloads for the installers and binaries for the .NET Core SDK.  Scroll down to the bottom and find the section for *Step-by-step instructions*.

![](https://storage.googleapis.com/ps-dotnetcore/stepbystep.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 2 - Step by step instructions**

From here click on the link for your operating system.  Then on the next page follow the instructions to install the .NET Core SDK.  There are also instructions to test your new installation.  Follow these as well.  You now have the .NET Core SDK installed on your machine.

### Visual Studio Code
This one is easy.  Just go to http://code.visualstudio.com and download the installer for your operating system.

![](https://storage.googleapis.com/ps-dotnetcore/codehomepage.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 3 - The Visual Studio Code home page**

##### Visual Studio Code Extensions
This guide relies upon two extensions to Visual Studio Code, the first is the C# extension.  To install it, with Visual Studio Code open, click on the bottom icon in the left sidebar to open the Extensions panel.

![](https://storage.googleapis.com/ps-dotnetcore/vscodeextensions.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 4 - The Extensions panel**

In the Extensions panel, at the top, type 'C#' in the search bar and press the enter key.  The top result should be for the C# extension published by Microsoft.  Press the green Install button to install the extension.  After that, repeat by searching for the 'mssql' extension and installing it.  To finish the installations, press the Reload button to reload Visual Studio Code.

![](https://storage.googleapis.com/ps-dotnetcore/vscodecsextension.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 5 - The C# extension**

The C# extension will provide a developer experience closer to that of Visual Studio for Windows with syntax highlighting and Intellisense along with debugging support.  The mssql extension will turn Visual Studio Code into a client for Microsoft SQL Server.

### Docker

If you already have access to a Microsoft SQL Server database, you can skip this section although if you want the true cross platform effect of ASP.NET Core, I recommend you follow along.  

Docker is a container host which virtualizes applications.  It is simiiar to a virtual machine, but more lightweight.  The application that this guide uses Docker to virtualize will be Microsoft SQL Server for Linux.  Yes, you read that correctly.  Microsoft SQL Server now supports Linux.  

To get Docker, go to the Docker home page at http://www.docker.com and under the *Get Docker* link at the top of the page, click on the link for your operating system.  Docker runs natively on Linux but there are environments for running containers on Windows and macOS and that is what the installers provide.  After the installation is complete, click on the Docker icon (in the system tray on Windows and the menu bar on macOS) and select the Kitematic menu item.  This will prompt you to install Kitematic, a GUI for Docker on Windows and macOS.  (FYI: Like Visual Studio Code, Kitematic is also an Electron application.)

With Kitematic installed and open, search for 'mssql' in the search bar.  Then find the item for 'mssql-server-linux' and click the Create button.

![](https://storage.googleapis.com/ps-dotnetcore/kitematicmssql.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 6 - The C# extension**

This will begin the download of the container image for SQL Server for Linux.

![](https://storage.googleapis.com/ps-dotnetcore/kitematicgetmsqlserver.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 7 - Downloading SQL Server for Linux**

After the image is downloaded it will create a new instance that is ready to run.  Before it can run a few settings must be changed.  First two environment variables need values.  To create them, click on the Settings tab in the upper right corner.  Under the General tab in the settings, in the secton Environment Variables, add a new environment variable with the key of 'ACCEPT_EULA' and a value 'Y'.  Then click the plus button on the right to create another variable with a key of 'SA_PASSWORD' and a value of a strong password to use for the 'sa' user.  Then click the Save button.

![](https://storage.googleapis.com/ps-dotnetcore/mssqlenvvars.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 8 - Creating Docker environment variables**

This should start the container.  If not, just click the start button.  Next, click the Hostname/Ports tab.  In this page, Docker will show mappings between ports on the container and the container host, which is the local machine.  The port for SQL Server is 1433.  Docker will map this port from the container to a random port on the local machine, 32773 on my machine.  To make things easier, I am going to change this to map to 1433 on the local machine.  Just double-click on the published port, change it to 1433 and then click the Save button.

![](https://storage.googleapis.com/ps-dotnetcore/kmports.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 9 - Mapping the SQL Server port**

That's all for the setup and installation.  Now it's time to create an ASP.NET Core application.

# The `dotnet` tool

You've already seen the `dotnet` tool briefly if you completed the steps to test the .NET Core SDK installation.  A lot of the tasks involved with managing a .NET Core application are performed with the `dotnet` tool.  It's a command line tool that can be extended with NuGet packages and we'll be using several of them in this guide.  For now, head back over to Visual Studio Code and open the integrated terminal window by going to View -> Integrated Terminal in the menu bar.  This will slide open a pane at the bottom on the editor with a shell prompt.

![](https://storage.googleapis.com/ps-dotnetcore/vscodeterm.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 10 - Visual Studio Code Integrated Terminal**

The shell will depend on the operating system used.  On macOS, it will default to the bash shell.  On Windows, you could be using Powershell.  Next I've created a directory to store my .NET Core applications called 'dotnetcore' so I'll change to that directory.  Then I'll run the follow `dotnet` command

```
dotnet new -h
```

The output is shown in the following figure:

![](https://storage.googleapis.com/ps-dotnetcore/vscodetmpls.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 11 - `dotnet` templates**

You've already seen the 'console' template.  This time we are going to use a web application template.  The `dotnet` tools provides several to choose from.  First you can create starter ASP.NET Core MVC application with some sample content.  You can also create web applications with skeletons supporting Angular or React.js.  And you can create an API as well.  For this guide, I am going to start from scratch, so I will pick the most minimal template that there is, the 'web' template.  The application will be a (very simple) e-commerce app that I call 'CoreStore' and I'll create the project with the following command in the terminal window:

```
dotnet new web -o CoreStore
```

This will create a new project called CoreStore in a new directory called CoreStore because of the -o (for output) option.  Next I want to open this project in Visual Studio Code.  The top icon in the left sidebar is the Explorer which shows the file structure of a project.  I'll click it and then click the Open Folder button and navigate to the directory my project is in.

![](https://storage.googleapis.com/ps-dotnetcore/vscodeexplorer.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 12 - Visual Studio Code Explorer**

You may see some output if this is the first time opening a project in Visual Studio Code.  This is just the C# plugin installing some needed software that we will use later on.  Also, you should see the following prompt at the top of the window:

![](https://storage.googleapis.com/ps-dotnetcore/vscodeassetsprompt.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 13 - Debug Assets prompt**

Visual Studio Code (via the C# extension) is asking to add some assets to enable debugging from within the editor.  This will happen the first time a new project is opened.  I'll click yes to allow this.  Notice that in the Explorer a new folder named '.vscode' has been added.

At this point, we have a complete (albeit extremely simple) web application that is ready to run.  To prove this, go to Debug -> Start Debugging in the menu bar.  The application will be built, the application will start on a local web server running on port 5000 and the default browser will open to the root of the application.

![](https://storage.googleapis.com/ps-dotnetcore/helloworld.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 14 - An Empty ASP.NET Core web application**

The result is the obligatory 'Hello World' application.  While this is progress, it's far from useful.  Let's see what makes this application tick.  The project structure of an ASP.NET Core application is a little different from past versions.  In the Explorer is a Program.cs file and a Startup.cs file.  These files control the launch and configuration of the application.  Open the Program.cs file by either clicking on it in the Explorer or by using the keyboard shortcut Cmd-P (Ctrl-P on Windows).  This will open a search bar at the top of the window.  Start typing the name of the file (Program.cs) and Visual Studio Code will filter the files in the project highlighting the best match.  Then select Program.cs from the list to open it.

![](https://storage.googleapis.com/ps-dotnetcore/vscodefileopenb.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 15 - Searching for Program.cs**

I'm not going to labor over this file too much other than to say that this is the entry point for the application as evidenced by the `Main()` method.  That method calls the following method:

```csharp
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>()
           .Build();
```

Note the `UseStartup()` method which is generic on the `Startup` class.  The `Startup` class contains the configuration for the application.  Open it in the file Startup.cs.

There are two important methods in this class.  First is the `Configure()` method.  This is where the heavy lifting of the configuration occurs.  The other is the `ConfigureServices()` method.  This method is a little misleading.  It is used to register services with the ASP.NET Core built-in dependency injection container.  And we'll be using it later when we get to the database.  Going back to the `Configure()` method, it's obvious where the 'Hello World' text is coming from.  The call to `App.Run()` explicitly creates the string.  For web applications, namely ASP.NET, we are accustomed to a framework like ASP.NET MVC to abstract some of these details for us.  And ASP.NET Core is no exception.  It's just that the empty web template doesn't set MVC up for us which is OK because it gives us the excuse to see the plumbing in action.

# Setting up ASP.NET Core MVC

Adding ASP.NET Core MVC to an ASP.NET Core app is pretty straightforward.  And a lot of it is similar to how ASP.NET NVC works in previous versions.  The main difference is going to be in setting up the framework in the Startup class.  So, in the `ConfigureServices()` method, put the following in the body (which is currently empty):

```csharp
services.AddMvc();
```

This will add the MVC framework to the DI container making it available for use in the application.  The next step is to set up routing information for the application which I'll do in the `Configure()` method.  Here I'll replace the call to `App.Run()` with the following:

```csharp
app.UseMvc(
    routes => {
        routes.MapRoute(
            name: "Default",
            template: "{controller=Catalog}/{action=Index}/{id?}"
        );
    }
);
```

If you are familiar with ASP.NET MVC then this code should make sense.  A route is a logical mapping between a URL and application code.  That URL is defined by the template.  The template in this is for a URL that has three components, the first two are required.  The first one will be mapped to a controller, the second to an action, and the last to an optional id.  In case the first or second components are omitted, default values are provided.  

The code mapped to this route is determined by the default ASP.NET Core MVC project structure.  A controller is a C# class that derives from a `Controller` base class which we will create soon.  The action is merely a method in the controller class.  The id, if included, will be passed to the action method as a parameter.

Before creating the controller class, I want to point out that there is another way of setting up a route which is quicker, but less flexible.  Instead of the `UseMvc()` method, the `UseMvcWithDefaultRoute()` method will do the same thing, except with an implied default route.  This route is the equivalent of the template:

```
{controller=Home}/{action=Index}/{id?}
```

# Controllers, Actions and Views
Right now, the application will still build and run but all requests will be met with an HTTP 404 'resource not found' error.  This is because ASP.NET Core MVC is looking for a controller and none exist, so the 404 is valid.  Like previous version of the MVC framework, controller live by default in a Controllers folder in the project.  In Visual Studio Code I'll create one using the Explorer.  This can be done by right clicking in the Explorer and choosing New Folder or by highlighting the location of the new folder and then in the bar on top on the file structure, clicking the New Folder icon:

![](https://storage.googleapis.com/ps-dotnetcore/vscodenewfolder.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 16 - Creating a new folder in the Visual Studio Code Explorer**

Next I'll create a new file, CatalogController.cs, in the Controllers folder by either right clicking on the Controllers folder and selecting New File or by highlighing the Controllers folder and clicking the New File icon.  Inside the new CatalogController.cs file, I'll create the following `CatalogController` class.  (By convention, controller classes have the 'Controller' suffix but the name of the controller omits it.  So the 'Catalog' controller lives in the `CatalogController` class.)

```
namespace CoreStore.Controllers
{
    public class CatalogController: Controller
    {
        
    }
}
```

Notice that as you type, Visual Studio Code provides Intellisense, suggesting the namespace in this example:

![](https://storage.googleapis.com/ps-dotnetcore/vscodecc.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 17 - Visual Studio Code Intellisense**

But there is an error!  Visual Studio Code has added a red line underneath `Controller`, the base class for all ASP.NET Core MVC controllers.  This is because the `Controller` class lives in the `Microsoft.AspNetCore.Mvc` namespace which has not been included in the CatalogController.cs file.  Before you start typing, take a look at the left of the line with the error,

![](https://storage.googleapis.com/ps-dotnetcore/vscodehint.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 18 - Quick Fix**

See the light bulb?  This means that Visual Studio Code can provide a 'Quick Fix' for this error.  To access the possible solutions click the light bulb and select an option from the popup menu.  However, some developers find it faster to use a keyboard shortcut.  With the cursor over the code causing the error, press Cmd-. (Ctrl-. on Windows) to access the popup.

![](https://storage.googleapis.com/ps-dotnetcore/vscodequickfix.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 19 - Quick Fix options**

The solutions available depend on the context.  In this example, Visual Studio Code has recognized that there is a `Controller` class in the `Microsoft.AspNetCore.Mvc` namespace.  It offers to add a using statement for that namespace or to prefix the `Controller` class with the namespace for the fully qualified name.  It also offers to generate a new class with the name `Controller`.  It even recognizes that there is a `Controllers` namespace and a `CatalogController` class both of which are very similiar to `Controller` in case I made a spelling error.  In this case, I'll select the first option, add the missing namespace, and correct the error.

Now I can add an action method to the class.  An action method must be public.

```
public string Index()
{
    return "Catalog Controller, Index Action";
}
```

At this point, we have enough to satisfy the default route of the application.  Start the application again by going to Debug -> Start Debugging in the menu bar or by pressing F5.

![](https://storage.googleapis.com/ps-dotnetcore/ccia.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 20 - The CatalogController in action**

Again, this is progress.  We've got more flexibility in the structure of our application thanks to the MVC framework, but no more flexibility in how content is created.  The result of the application is still just plain text.  And while I could add some HTML tags to the return value of the `Index()` method, this practice won't scale.  Instead, we need to be able to write our HTML external from the controller and dynamically show the correct markup for the request.  This is where views come in.

A view in ASP.NET Core MVC is a file which is a combination of HTML and C# in a special syntax called Razor.  Views live in a folder called Views in the root of the project.  Each controller has a subfolder and in that subfolder lives a view file for each action method.  The view files have an extension of .cshtml.  The action method in the controller, can return a call to the `View()` method which will by default, look in the Views folder, in the controller folder, for a view file named the same as the action.  So let's set that up right now. 

I'll click in the explorer to ensure I'll create a new folder in the root of the project and then click the New Folder icon and call it Views.  Then with Views selected, I'll create another new folder called Catalog.  (Again omitting the 'Controller' suffix.)  In the Catalog folder, I'll create a new file called Index.cshtml and open it in Visual Studio Code.

![](https://storage.googleapis.com/ps-dotnetcore/vscodevies.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 21 - Structure of the Views folder**

While eventually this will have both C# and HTML, for now I'll just add some simple HTML.

```html
<h1>Catalog Controller, Index Action</h1>
```

Open up CatalogController.cs.  In the `Index()` method, change the return type to `IActionResult` and return a call to `View()`:

```
public IActionResult Index()
{
    return View();
}
```

Now run the application again (press F5) and see the results:

![](https://storage.googleapis.com/ps-dotnetcore/vscodehtml.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 22 - Loading a view**

Ah!  Now we're getting somewhere!  This is **much** more flexible and managable.  For the sake of demonstration, add the following code to the view:

```html 
<ul>
@foreach (var i in new int[] {1, 2, 3, 4, 5})
{
    <li>@i.ToString()</li>
}
</ul>
```

And run the application again.

![](image goes here)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 23 - Using Razor**

This introduces the advantage of Razor.  Notice that I am able to effortlessly switch between markup and code just by prefixing the C# statements with an @ sign.  Obviously, you would never do anything like this in production but I wanted to show an example before moving on to ...

# Models

Any useful web application must handle data.  To abstract the details of accessing different databases, .NET Core offers an Object Relational Mapper (ORM), Entity Framework Core.

### Entity Framework Core

If you are familiar with previous versions of Entity Framework, EF Core will not be much different in terms of code.  Some of the tooling is different though.  First, I'll start off by adding Entity Framework Core to the project.  It lives in a NuGet package named `Microsoft.EntityFrameworkCore`.  Depending on the database you wish to use, you'll need to install a package for that database.  Generally, we don't use the same kind of database for local development as in production.  Previously, SQL Server Local DB was the usual choice.  However, it only runs on Windows.  And in the past that was OK because everything ASP.NET ran on Windows.  With ASP.NET Core, we can no longer safely make that assumption.  So I'll turn to another local database, SQLite.

SQLite is a file based database that runs locally.  It is typically used for development and not in production because of scaling issues.  But it does replicate the behavior of a relational database very well.  Thus it's a great choice for development.  And thanks to EF Core, the differences between SQLite and a production database (I'll be using SQL Server for Linux) are abstracted so the code for the two is close to identical!

To get started with SQLite, the NuGet package for EF Core and SQLite needs to be added to the project.  This is done via the `dotnet` command:

```
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

After this command is done, look inside of the CoreStore.csproj file and notice the new reference:

```
<PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="2.0.0" />
```

With EF Core added to the project, I'm ready to start writing model code.

# Model and Context Objects

The model object will represent the logical structure of the data for the application in a C# class.  ASP.NET Core MVC, unlike the controllers and views, doesn't have a predefined opinion of where these live in the project structure or if they live in the project at all.  It's common to have the model code in a separate assembly in larger projects.  For this demo, we can easily keep everything in the same project and so I'll create a Models folder for the model code.  And inside of that folder I'll create a file Product.cs with the following code:

```
namespace CoreStore.Models
{
    public class Product
    {
        
    }
}
```

This is just a POCO, a 'Plain Old C# Object'.  And it will have plain old C# properties to represent the data stored.  And here we are presented with an excellent chance to show off some more Visual Studio Code tooling.  Just like its big sister, Visual Studio Code support snippets or templates to generate boilerplate code.  C# properties generally have the same syntax and so Visual Studio Code has a snippet that creates them more quickly.  And you can add your own too.  I'll stub out a C# property in the Product class with a snippet by typing 'prop' and pressing the TAB key.

![](https://storage.googleapis.com/ps-dotnetcore/vscodepropsnip.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 24 - Using the 'prop' snippet**

This snippet has generated an auto-implemented C# property.  It assumes the public access modifier and highlight the type.  At this point I can change the type or press the TAB key again to accept `int`.  And that's what I want to do because this property will the ID of the product.  Now the default name is highlighted.  I'll change the name to `ID` and hit TAB once more to go to the end of the line thus completing the property.  I'll add two more properties for now to finish out the class:

```
namespace CoreStore.Models
{
    public class Product
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}
```

One more thing in the model object, I want the database to supply the value for the ID to ensure that it will be unique.  For this, I'll add the `DatabaseGenerated` attribute to the `ID` property.  This attribute is located in a namespace (`System.ComponentModel.DataAnnotations.Schema`) that is not used in the Product.cs file, but I can add it with the quick fix.  The attribute takes a parameter to tell the database how to generate the value.  I'll use `DatabaseGeneratedOption.Identity` to have it generate sequential numbers.  The `ID` property looks like this now:

```
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public int ID { get; set; }
```

And for the moment, that's all for the model class.  Now we need a way to marshal the model class to the database itself.  EF Core does this via a context, which is derived from a base `DbContext` class.  I'll create one in a new file, called StoreContext.cs, in the Model folder:

```
using Microsoft.EntityFrameworkCore;

namespace CoreStore.Models
{
    public class StoreContext: DbContext
    {
        
    }
}
```

The context class needs a constructor which takes a parameter of type `DbContextOptions`.  This is generic on the context type, `StoreContext`.  The constructor is empty, but it needs to pass the options object to the base constructor.  Here is the code: (FYI - there is a 'ctor' snippet to generate a constructor)

```
public StoreContext(DbContextOptions<StoreContext> options): base(options) { }
```

Next I'll create a `DbSet`.  This object will be how we access the actual model object instances populated with data from the database.  This is a property of the context class:

```
public DbSet<Product> Products { get; set; }
```

That's all that is required for now.  There is an optional step which I will go ahead and show here.  In a few minutes, I'll show how to generate a database for this context and model object.  By default, EF Core will create a table for the model object using the same name, Product in this case.  However, sometimes you might want to use different names.  To do this, override the OnModelCreating method in the context class:

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>().ToTable("Products");
}
```

This is straightforward using the `modelBuilder` parameter to map the `Entity<>` for the `Product` class to a table called 'Products'.  Again this is optional as by default EF Core would have created a table called 'Product'.

That's all, for now, for the context class.  There is one more step that I am going to take and it's out of convenience.  I'll throw this code out later when we look at migrations.  But this will help us get data into the database faster.  I'm going to write a static class to seed the database with some sample data that we can display in ASP.NET Core.  In the Models folder, create a new file, DbSetup.cs:

```
using System.Collections.Generic;

namespace CoreStore.Models
{
    public static class DbSetup
    {
        public static void Setup(StoreContext ctx)
        {
            ctx.Database.EnsureCreated();

            if (ctx.Products.Count() == 0)
            {   
                ctx.Products.AddRange(new List<Product> {
                    new Product { Name = "Apple", Price = 1.50m },
                    new Product { Name = "Banana", Price = 2.00m }
                });

                ctx.SaveChanges();
            }
        }
    }
}
```

This is an example of how to use the context class.  The first line in the `Setup()` method makes sure that a database exists and creates one if not.  We'll get to the specifics of which database soon.  Next, the `Products` has a method `AddRange()` which accepts one or more `Product` objects and adds those to the `DbSet`.  To commit the change to the datbase, the `SaveChanges()` method is called on the context.  Note that before adding the sample data, I check to make sure that `Products` has no objects.  This avoids adding duplicate sample data every time the object is run.  Later on, we'll see a better way to do this.

Now we need to call this method from somewhere.  Application startup would be a good time so in the `Configure()` method in the `Startup` class I'll add a call:

```
public void Configure(IApplicationBuilder app, IHostingEnvironment env, StoreContext ctx)
{
    // existing code here

    DbSetup.Setup(ctx);
}
```

Notice that the signature of the `Configure()` method has been changed.  It now accepts a `StoreContext` which is passed to `DbSetup.Setup()`.  But where does this `StoreContext` come from?  This is where the built-in dependency injection for ASP.NET Core comes in useful.  Assuming that `StoreContext` is registered with the DI container, ASP.NET Core will create one when calling the `Configure()` method.  That means we need to register it.

Remember that DI registration is done in the `ConfigureServices()` method.  To register a database context, there is a special method.  Here is the code:

```
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddDbContext<StoreContext>(options => {
        options.UseSqlite("Data Source=store.db");
    });
}
```

The `AddDbContext()` method is generic on the context to add, `StoreContext` in this code.  It also uses a `DbOptionsContextBuilder` to configure the context.  The details of this configuration depnend on the database.  In this application, I want to use SQLite, so I call the `UseSqlite()` method and pass it a connection string which is just a path to the location of the SQLite database file.

Next I'll return to the `CatalogController` class and add some logic to pull data from the database.  This will be done via a context object so I'll create a class level variable of type `StoreContext` in `CatalogController`.  Next I need some way to initialize the context object.  I'll do that in a constructor, using the DI container to inject an instance of the context as a parameter and then use that paramter to initialize the variable.  Finally, in the `Index()` method, I'll just get all of the product objects from the context.  Here's what that looks like so far:

```
public class CatalogController: Controller
{
    private StoreContext ctx;
    
    public CatalogController(StoreContext _ctx)
    {
        ctx = _ctx;
    }

    public IActionResult Index()
    {
        var products = ctx.Products.ToList();
        return View();
    }
}
```

To get the products to the view, just pass them to the `View()` method.  So the line:

```
return View();
```

is now

```
return View(products);
```

The final step in this part is to tell the view to expect a `List<Product>`.  In the Index.cshtml file, add this directive at the top:

```
@model IEnumerable<CoreStore.Models.Product>
```

This turns the view into a strongly-typed view.  In other words, it will have a model object that is of type `IEnumerable<CoreStore.Models.Product>`.  Inside of the view, I can then iterate over the `@Model` (with a capital 'M') to display the individual products:

```
<table>
    <tr>
        <th>Name</th><th>Price</th>
    </tr>
    @foreach (var product in @Model)
    {
        <tr>
            <td>@product.Name</td><td>$@product.Price.ToString("F2")</td>
        </tr>
    }
</table>
```

Running the application yields this result:

![](https://storage.googleapis.com/ps-dotnetcore/modelview.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 25 - Data in the view**

# More Views

This is the point that we have been aiming for.  The application is technically 'data-driven' but it is still not complete.  Right now the application has only a single view that pulls a fixed set of data from the database.  A real world application would support the CRUD operations with CRUD an acronym for 'Create Read Update Delete'.  While I could code all of this functionality by hand, a lot of the code would be boilerplate.  In the next CRUD application, I'd have to write it all over again.  Instead, thanks to EF Core and a NuGet package that adds some extra commands to the `dotnet` tool, we can generate this boilerplate code.

First, two NuGet packages have to be added to the project.  Using the `dotnet add package` command add a reference to `Microsoft.VisualStudio.Web.CodeGeneration.Deisgn`.  If prompted to restore the packages, go ahead.  The second NuGet package is the extension to the `dotnet` CLI tool.  With extensions to the `dotnet` tool, we must manually update the .csproj file and restore the packages by hand.  Fortunately, it's not that hard but always take care when modifying the .csproj file.

In Visual Studio Code, open the CoreStore.csproj file.  Just before the closing `</Project>` tag, add a new `<ItemGroup>` tag and close it.  Then add a reference to `Microsoft.VisualStudio.Web.CodeGeneration.Tools`.  But this will not be a `<PackageReference>` tag.  Instead it will be a `<DotNetCliTooReference>` tag.  Here is the code to add:

```
<ItemGroup>
  <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0"/>
</ItemGroup>
```

After saving this file, the packages need to be restored to download the NuGet package with the tooling.  Had we added a package reference, Visual Studio Code would have prompted for the restore.  But with a CLI tool package, it has to be done manually.  There are two ways to do this.  First is in the terminal window.  Run

```
dotnet restore
```

at the prompt.

The second way is to run it within the Visual Studio Code command palette.  Press Cmd-Shift-P (Ctrl-Shift-P on Windows).  At the top of the window the command palette will appear.  Type 'restore' and then select the '.NET: Restore Packages' option.  This is do the same thing as running `dotnet restore` in the terminal window.

![](https://storage.googleapis.com/ps-dotnetcore/vscoderestorecp.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 26 - Restoring the packages**

Now in the terminal window, a new command is available with the `dotnet` tool:

```
dotnet aspnet-codegenerator
```

![](https://storage.googleapis.com/ps-dotnetcore/aspnetcodegen.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 27 - Running the ASP.NET Core code generator**

As shown in the output, this command lets you generate code for controllers, views and other project assets.  I'm going to use the code generator for the controller.  In cooperation with the context class from EF Core, the code generator will generate a new Controller with actions and views to list, create, read, update and delete products in the database.  First I'll, delete the existing `CatalogController` and associated Catalog views folder.  Then I'll run the following command (it's a *long* one)

```
dotnet aspnet-codegenerator controller --project CoreStore.csproj --controllerName "CatalogController" \
    --dataContext "StoreContext" --model "Product" --controllerNamespace "CoreStore.Controllers" \
    --relativeFolderPath "Controllers"
```

This will generate the CatalogController.cs file in the Controllers folder.  It will also create a Catalog folder in the View folder and generate a .cshtml view file for each of the Create, Delete, Edit, Details (Read) and Index (List) actions.  Before looking at the results, let's look at the command.

There are a lot of options here.  I used the long form to be clearer but generally would use the short form.  You can see those by running the command:

```
dotnet aspnet-codegenerator controller -h
```

Here is a brief explantaion:

- project - the relative path to a .csproj file
- controllerName - the class name of the new controller
- dataContext - the name of the data context class of the model
- model - the name of the model object class
- controllerNamespace - the fully qualified namespace to put the new controller in
- relativeFolderPath - the relative path of the folder to put the new controller in

Anything that does not already exist will be created.  And now to look at the output.  Press F5 one more time.

![](https://storage.googleapis.com/ps-dotnetcore/gencode.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 28 - Running the generated controller**

There is some good news and bad news here.  First the good news.  The application didn't crash so the generated code didn't break anything seriously.  And our data still shows up.  But what about the text for 'Create New' and 'Edit'?  It seems like those should be links.  While we can get to the Create view by navigating to `/Catalog/Create', the text boxes have no labels and submitting the form does nothing.

![](https://storage.googleapis.com/ps-dotnetcore/createproduct.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 29 - The Create view .. sort of**

Let's take a look at the code and see if it gives any clues as to what is going on.  First open the Index.cshtml view.  Just after the opening `<body>` tag is a paragraph and inside of it, this code:

```
<a asp-action="Create">Create New</a>
```

This is an anchor tag, with a twist.  Instead of an href attribute, there is this asp-action attribute.  This is what is called a tag helper.  This particular tag help will generate a URL for the action in the value based on the current controller.  But it's not doing that.  In fact, if you do a view source on the HTML in the browser, you'll see that the anchor tag is returned to the browser verbatim as it appears in the view file.  The browser doesn't know what to do with the tag helper so it bails.  This is also the reason why the other links don't work, why the create form has to labels and why submitting it does nothing.

It took me a while to find the solution to this when I first saw it.  But it makes more sense when you realize that tag helpers are part of ASP.NET Core MVC, not just ASP.NET Core in general.  So they are only useful if your application includes the MVC framework.  The template we used to create the application was an empty ASP.NET Core application without MVC support.  So it makes no sense to include tag helpers with that.  (The mvc template does include support for tag helpers.)  Forunately, the solution to this is simple.  In the Views folder, create a new file called _ViewImports.cshtml.  This file will essentially be executed before every view file is rendered.  So it's a good place to include the tag helper support.  This is the only line needed in _ViewImports.cshtml:

```
@addTagHelper *,Microsoft.AspNetCore.Mvc.TagHelpers
```

This line adds support for all of the built in tag helpers in `Microsoft.AspNetCore.Mvc.TagHelpers` to every view.  With this in place run the application again.  The links will all be working.  The Create view will have labels for the text boxes.  And the form will submit data to be stored in the database.  The Edit, Details and Delete views also work.  Check it out.

# Migrations

The application is really coming along.  However, the database structure is very rigid.  What if it needs to change?  Right now, we have no way to keep track of inventory.  How would we add such data to the database?  This currently would be messy.  But EF Core has a tool that helps make it easier.  Meet migrations.

A migration is basically a log of the changes in the structure of a data model.  In EF Core these changes are stored as C# code.  When executed, this code will commit the changes to the structure of the database.  Each time you make a change in the structure of the data model, say adding a stock level property to the `Product` class, you create a migration.  That migration contains the code needed to add that new property as a column in the database.  It also has code to remove it so you can walk backward in the migration history as well as forward.

### More CLI tooling

The `dotnet` tool once again must be extended to take advantage of EF Core migrations.  Like the ASP.NET Core code generator, this NuGet package reference must be added by hand.  Open the CoreStore.csproj file and before the closing `</Project>` tag add the following code:

```
<ItemGroup>
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
</ItemGroup>
```

Restore the packages using either the `dotnet restore` command or by running the restore command from the command palette.  After that, you will have access to the `dotnet ef` command.  There are three commands in the `dotnet ef` command.  One of them is for migrations but before looking at it, we need to start with a clean slate.  Run the command:

```
dotnet ef database drop
```

and confirm that you want to delete the database file.  Now normally this is not something that would happen in the real world.  Usually a project would plan for migrations at the outset.  But because of the non-linear path the learning process takes, we need to remove the test database from the previous sections.  

Also, remove the call to `DbSetup.Setup()` in the `Startup` class.  We'll be using migrations to seed the database later on.  Next in the terminal window, run `dotnet build` to manually build the application to make sure that everything is up to date.  Again, this is not something that would be done in a real world application that used migrations initially.

Before creating a migration, a few changes have to be made to the `StoreContext` class that will be required to seed the database later on.  A discussion of why these are required is outside the scope of this guide.  But the class will require an empty parameterless construction and an override of the `OnConfiguring()` method.

```
public StoreContext() { }

// existing code omitted

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlite("Data Source=store.db");
}

```

Now to create the first migration.  The command to create one is:

```
dotnet ef migrations add
```

followed by the name of the migration.  Usually, the first migration is named 'Initial'.

```
dotnet ef migrations add Initial
```

This will find the data context class and create a migration fram it that will add a table for the `Product` model class.  Look in Visual Studio Code at the Explorer.  A Migrations folder has been added with three files.  The first two are created specific to the migration.  The name is a timestamp followed by the migration name.  The last is a log of the migrations created.  The one that contains the code we care about is the first one, the file without 'Designer' in the name.  Open it in Visual Studio Code.

The migration is a C# class with two methods, `Up()` and `Down()`.  When the migration is applied to the database, the `Up()` method is called.  When the migration is rolled back, the `Down()` method is called.  The migrations are applied and rolled back with the `dotnet ef` command.  We'll do that in a minute.  First look at the body of the `Up()` method.  It has a method to create a table with a name of 'Products', just like the `StoreContext` class is configured to do.  It also adds a column in the table for each property in the `Product` class.  And the `ID` is the primary key which is generated sequentially, as a result of the `DatabaseGenerated` attribute applied to the `ID` property in the `Product` class.  So all of this will be run when the migration is applied.  The `Down()` method just undoes everything is the `Up()` method.  In this simple case it drops the table.

To apply the migration (which will also create the database since it doesn't exist) run:

```
dotnet ef database update Initial
```

The last parameter to the `update` command, the migration name, is optional.  If omitted the most recent migration will be applied.  If you look in the Explorer in Visual Studio Code, there is once again a 'store.db' file for the SQLite database.  I'm not going to bother looking inside the database yet as there currently no tools in Visual Studio Code for mangaging SQLite files.  We'll do this later when moving to SQL Server.  In the meantime, running the application at this point will not be very exciting as the database contains no data.  But we can also use migrations to seed the database.

Seeding the database first requires an empty migration.  I could create one manually but it's easier to easier to let `dotnet` do it.  Running the `dotnet ef migrations add` command when no changes have been made to the data model since the last migration was applied will generate an empty migration with just a blank `Up()` and `Down()` method which is exactly what I need.

```
dotnet ef migrations add Seed
```

Add the following code the to `Up()` method:

```
using (var ctx = new StoreContext())
{
    ctx.Products.AddRange(new List<Product> {
        new Product { Name = "Apple", Price = 1.50m },
        new Product { Name = "Banana", Price = 2.00m }
    });

    ctx.SaveChanges();
}
```

This code should look familiar.  In fact, I just copied the body on the `using` statement from the `DbSetup` class.  The difference is that in the migration the context is created in a different way. (Incidentally, this is why the new constructor and `OnConfiguring()` override had to be added to the context class.)

Don't forget that what goes up must come down.  Or in this case, what is done must have a way to be undone so add the following code to the `Down()` method:

```
using (var ctx = new StoreContext())
{
    ctx.RemoveRange(ctx.Products);
    ctx.SaveChanges();
}
```

This code just removes all the the products from the database.  Now I can seed the database by applying the migration:

```
dotnet ef database update Seed
```

Now running the application will show the data again.  But all we've done is essentially restore the application to the state that it was before migrations were brought in.  But now let's make a change to the data model.  Change the `Product` by creating a new property for inventory level:

```
public int StockLevel { get; set; }
```

And add a new migration:

```
dotnet ef migrations add StockLevel
```

Look at the `StockLevel` migration file.  The `Up()` method adds a column for 'StockLevel' to the 'Products' table.  The `Down()` method removes it.  The rest of the database is untouched.  This is the first part of the power of migrations.  By specifying the name of the migration I can revert to any point in the history of changes in the database.  If you recall, we have existing data in the 'Products' table.  What happens when the 'StockLevel' column is added?  In the `Up()` method, the `AddColumn()` method has a default value which is set to 0 right now.  I'm going to set it to 100 instead.  That will also be the value for new products as well so I need to make that change.

In the `CatalogController` class, find the `Create()` method.  There are two of them.  The first one, marked with the `HttpPost` attribute, is called when the a request is made to display the form.  The second one, marked with the `HttpPost` attribute, is called when the form is submitted.  THe second one needs to be modified.  Just before the new product is added to the context:

```
_context.Add(product);
```

Set the `StockLevel` to 100:

```
product.StockLevel = 100;
```

So the entire method is:

```
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create([Bind("ID,Name,Price")] Product product)
{
    if (ModelState.IsValid)
    {
        product.StockLevel = 100;
        _context.Add(product);
        await _context.SaveChangesAsync();
        return RedirectToAction(nameof(Index));
    }
    return View(product);
}
```

Also, change the details view to show the `StockLevel`.  In Details.cshtml, add:

```
<dt>
    @Html.DisplayNameFor(model => model.StockLevel)
</dt>
<dd>
    @Html.DisplayFor(model => model.StockLevel)
</dd>
```

before the closing `</dl>` tag.  This should be enough to see the effect of the migration.  Apply the migration:

```
dotnet ef database update StockLevel
```

Run the application again.  At first everything seems to be the same.  But click in the 'Details' link for a product:

![](https://storage.googleapis.com/ps-dotnetcore/stocklevel.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 30 - After applying the StockLevel migration**

The `StockLevel` property is now recognized.  Trying adding a new product and view its details.  The `StockLevel` property is also handled correctly.

At this point, if the `StockLevel` property needed to be removed that could be done by 'updating' the database to a previous migration.  For example:

```
dotnet ef database update Seed
```

would call the `Down()` method on the StockLevel migration, remove the property and revert the database to the state it was after applying the Seed migration.  In fact, while writing this guide I had to do that.  I forgot to change the default value of `StockLevel` to 100 in the migration and when I applied it, all of the existing products had no stock!  So I reverted back to the Seed migration, changed the default value of `StockLevel` and then re-applied the StockLevel migration.

# Adding a Simple API
