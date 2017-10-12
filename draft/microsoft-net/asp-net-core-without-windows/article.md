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


