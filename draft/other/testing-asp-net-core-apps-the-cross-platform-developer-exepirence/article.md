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

Go ahead and open the folder in Visual Studio Code now.  This will prompt to add build and debug assets.  To ensure that they are properly configured for the web project, this must be done before adding the test project.

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

This last step is curious.  Currently, Visual Studio Code does not provide Intellisense for referenced projects unless the editor is restarted.  I'm not sure what causes this but fortunately the solution is simple.  Just open the Command Palette with Cmd-Shift-P on macOS or Ctrl-Shift-P on Windows.  In the search bar, type `reload` and select the `Reload Window` option.

Figure 4. Reloading the editor

# The First Test

The `dotnet` CLI added a basic test file when the project was created.  It's in the `UnitTest1.cs` file.  I'll start off by renaming this file to `ControllerTests.cs`.  Then I'll open `ControllerTests.cs` and rename the `UnitTest1` class to `ControllerTests`.  The class itself does not need to be marked explicitly as containing unit tests.  The tests themselves are marked with a `[Fact]` attribute.  The `dotnet` CLI has created one of these for me.  I'll rename the test to something more descriptive, `VerifyIndexViewType`.  The `ControllerTests` class now looks like this:

```
public class ControllerTests
{
    [Fact]
    public void VerifyIndexViewType()
    {
    }
}
```

This test will create a new instance of the `HomeController` class, call the `Index` method and then check the return type which should be `ViewResult`.

```
var controller = new HomeController();
var result = controller.Index();
Assert.IsType<ViewResult>(result);
```

Figure 5. Errors in the editor

This will generate a new a red lines which mean errors.  So I'm going to fix those next.  The first one, under `HomeController` is easy to correct.  Putting the cursor on the `HomeController` text and pressing Cmd-. on macOS, Ctrl-. on Windows, will bring up the Quick Fix popup menu.  This will give the option to add the `CoreStore.Web.Controllers` namespace to the file.  In this case, fixing one error causes another.  This one is a little more subtle.  Hovering over the error will show the following.

Figure 6. The missing assembly

This error is complaining about the `IActionResult` type returned by the `Index` action.  `IActionResult` is in the assemblies for ASP.NET Core MVC which are currently not added to the test project.  In ASP.NET Core 2.0, all of the assemblies for ASP.NET Core are in a single NuGet package: `Microsoft.AspNetCore.All`.  I'll add that to the test project with the `dotnet` CLI.  Be sure to run this command from the test project folder in the integrated terminal.

```
> dotnet add package Microsoft.AspNetCore.All
```

This will trigger a prompt to restore the packages in the project as the new one has been added to the test project .csproj.

Figure 7. The restore prompt

Click `Restore` to permit this and restore the packages.  To get rid of the red line, the editor may need to be reloaded again.  The last error is for the `ViewResult` type in the `Assert` call.  It can also be resolved with the Quick Fix.

And that's the code for the first test.  So what does it mean?  Let's look at the three parts of a unit test and how they relate to `VerifyIndexViewType`.

* Arrange - This is where the prequisites for the test are created.  In this case all that is needed is an instance of the `HomeController`.
* Act - This is the action that is being tested by the unit test.  Here the action is the `Index` method.
* Assert - Here is where the result of the Act step is verified.  The assertion API is provided by xUnit.NET.  The `Assert` class has many methods that do a variety of checks.  The `IsType` method takes a generic type and then compares that to the type of the method parameter.  So in this case its checking to make sure that the type of the `result` is identical to `ViewResult`.

Notice that Visual Studio Code has added some links above the signature of the test method.

Figure 8. Test method links

Clicking on the `run test` link will build the `CoreStore.Web` project and the `CoreStore.Tests` project.  The it will look for unit tests in the test project.  Those tests will be run and the results reported.  And for this first run, my very simple test passed.  

Figure 9. All tests passed

Just so we can see what a failing test looks like I'll change the generic type in the `Assert` call to `string`.

```
Assert.IsType<string>(result);
```

Clicking the `run tests` link again will build and run the tests but this time the test failed.  As the summary states, the expected type of the `result` was a `System.String` and the actual type was `Microsoft.AspNetCore.Mvc.ViewResult`.  Thus the assertion falied.

# Adding the Model

Let's go deeper into the result of an action method.  The `ViewResult` has a `Model` property referring to the model object passed to the `View` method in the action method.  Right now, none of my action methods do this so I'll add a new one to the controller class.  But first, I need a model class.  So I'll add a `Models` folder to the `CoreStore.Web` project and then a new `Product.cs` file for the model object.

```
namespace CoreStore.Web.Models
{
    public class Product
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public demical Price { get; set; }
    }
}
```

Now I can add a new action method to pass a list as the model object to a view.

```
public IActionResult List()
{
    var products = new List<Product> {
        new Product { ID = 1, Name = "Apples", Price = 1.50m },
        new Product { ID = 2, Name = "Bananas", Price = 2.00m }
    };
    
    return View(products);
}
```

And the corresponding view in a new file `Views/Home/List.cshtml`.

```
@model IEnumerable<CoreStore.Web.Models.Product>

<ul>
@foreach (var product in @Model)
{
    <li>@product.Name</li>
}
</ul>
```

# Testing the Model

In `ControllerTests.cs` in `CoreStore.Tests` add a new test method.

```
[Fact]
public void VerifyListActionProductCount()
{
    var controller = new HomeController();
    var result = Assert.IsType<ViewResult>(controller.Index());
    var model = Assert.IsType<IEnumerable<Product>>(result.Model);
    Assert.Equal(2, model.Count());
}
```