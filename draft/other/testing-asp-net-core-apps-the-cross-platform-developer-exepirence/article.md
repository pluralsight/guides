# Introduction

Now that .NET Core, and ASP.NET Core, is a reality developers are having to reexamine the tools and methods of creating web applications.  This guide will summarize how to test a simple ASP.NET Core MVC application using cross platform tools such as the `dotnet` CLI tool and Visual Studio Code.  While a vast majority of the source code will be identical to a project written using Visual Studio on Windows, the goal is to focus on the process.  I'm assuming that you already have the .NET Core SDK, Visual Studio Code and the C# extension installed.  I'll also assume that you have some experience developing simple ASP.NET MVC (Core or otherwise) applications.  This guide is about testing, not development.

# Setup

I'm going to start from within Visual Studio Code.  With the integrated terminal window open, I'll change to my development directory and create a new directory for the demo project.  This will be the beginning of an e-commerce application so I'll call it `CoreStore`.

```
> mkdir CoreStore
> cd CoreStore
```

Next I'll create two projects, one for the web application and one for the unit tests.  The web application project will be named `CoreStore.Web`.

```
> dotnet new web -o CoreStore.Web
```

The test project required a choice.  Two testing frameworks are support by the `dotnet` CLI.  The first is MSTest, which is from Microsoft.  It's a fine product and if you have previous experience with testing Microsoft software in Visual Studio you've likely used it.  But now we are in a cross-platform world and while MSTest with .NET Core is cross platform, there are other options.  One of these is xUnit.NET.  This is an open source project that is inspired by the [xUnit concept](https://en.wikipedia.org/wiki/XUnit).  There are many frameworks that follow this structure and xUnit.NET is the one for .NET projects.  The `dotnet` template for xUnit is `xunit`.  I'll name the test project `CoreStore.Tests`.

```
> dotnet new xunit -o CoreStore.Tests
```

So the entire project structure should look like this when open in Visual Studio Code:

Figure 1. The project folder structure

### `CoreStore.Web` Setup

The empty ASP.NET Core web application has no support for MVC.  I could have created a skeleton MVC application from the `dotnet` CLI but it would have included a lot of plumbing and starter content which would just get in the way.  Instead, I'll add MVC by hand.  It's only a few lines of code.  

First in the `Startup.cs` file, in the `ConfigureServices` method, add the following line of the code:

```
services.AddMvc();
```

Then in the `Configure` method, remove the call to `app.Run` and replace it with the following:

```
app.UseMvcWithDefaultRoute();
```

This will be the same as creating an explicit route with a default `Home` controller, `Index` action, and optional `id`.  Therefore I need to create those resources.  First in the `CoreStore.Web` project root, I'll add a new folder named `Controllers` to follow the ASP.NET Core MVC convention.  Inside of the folder, I'll continue to honor the convention with a C# file named `HomeController.cs`.

Figure 2. The Controllers folder structure

The `HomeController` class will be very simple:

```
namespace CoreStore.Web.Controllers
{
    public class HomeController: Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

I'll add more to this class throughout the guide but just to get started I'll have a single action method which will return its default view.  So I'll need to create that.  In the `CoreStore.Web` project root I'll create a new folder called `Views` and in it a subfolder called `Home`.  In the `Home` folder I'll place a view file called `Index.cshtml`.  The view file, for now, will contain some simple HTML.

```
<h1>The Index Page</h1>
```

And here is the complete `CoreStore.Web` project structure in Visual Studio Code.

Figure 3. The `CoreStore.Web` project structure

### `CoreStore.Tests` Setup

The only task to setup the `CoreStore.Tests` project is to add a reference to the `CoreStore.Web` project that is the target of the tests.  In the integrated terminal, I'll change to the directory for the test project.

```
> cd CoreStore.Tests
```

Then use the `dotnet` CLI to add a reference to the `CoreStore.Web` project file.

```
> dotnet add reference ../CoreStore.Web/CoreStore.Web.csproj
```

Opening up the `CoreStore.Tests.csproj` file, notice the new `ProjectReference` tag.

```
<ItemGroup>
    <ProjectReference Include="..\CoreStore.Web\CoreStore.Web.csproj" />
</ItemGroup>
```

### Final Step

This last step is curious.  Currently, Visual Studio Code does not provide Intellisense for referenced projects unless the editor is restarted.  I'm not sure what causes this but fortunately the solution is simple.  Just open the Command Palette with Cmd-Shift-P on macOS or Ctrl-Shift-P on Windows.  In the search bar, type `reload` and 
