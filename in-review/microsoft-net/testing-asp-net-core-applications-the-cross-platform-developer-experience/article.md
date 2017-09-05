# Introduction

Now that .NET Core, and ASP.NET Core are increasingly used worldwide, developers are having to reexamine the tools and methods of creating web applications. This guide will cover how to test a simple ASP.NET Core MVC application using cross-platform tools such as the `dotnet` CLI tool and Visual Studio Code. While a vast majority of the source code will be identical to a project written using Visual Studio on Windows, the goal is to focus on the process. 

The code for the finished sample can be found on [Github](https://github.com/douglasstarnes/CoreStore).

## Prerequisites

I'm assuming that you already have the .NET Core SDK, Visual Studio Code and the C# extension installed.  I'll also assume that you have some experience developing simple ASP.NET MVC (Core or otherwise) applications. *This guide is about testing, not development.*

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

### Two Frameworks

The test project required a choice. The `dotnet` CLI supports two testing frameworks. The first is MSTest, which is from Microsoft.  It's a fine product, and if you have previous experience with testing Microsoft software in Visual Studio you've likely used it.  

But now we are in a cross-platform world and while MSTest with .NET Core is cross-platform, there are other options.  One of these is [xUnit.net](https://xunit.github.io/).  This is an open source project that is inspired by the [xUnit concept](https://en.wikipedia.org/wiki/XUnit). There are many frameworks that follow this structure and xUnit.NET is the one for .NET projects.  The `dotnet` template for xUnit is `xunit`.  I'll name the test project `CoreStore.Tests`.

```
> dotnet new xunit -o CoreStore.Tests
```

So the entire project structure should look like this when open in Visual Studio Code:


![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure01.png)

Figure 1. The project folder structure

### `CoreStore.Web` Setup

The empty ASP.NET Core web application has no support for MVC. I could have created a skeleton MVC application from the `dotnet` CLI, but it would have included a lot of plumbing and starter content which would just get in the way.  Instead, I'll add MVC by hand.  It's only a few lines of code.  

First in the `Startup.cs` file, in the `ConfigureServices` method, add the following line of the code:

```
services.AddMvc();
```

Then in the `Configure` method, remove the call to `app.Run` and replace it with the following:

```
app.UseMvcWithDefaultRoute();
```

This will be the same as creating an explicit route with a default `Home` controller, `Index` action, and optional `id`.  Therefore I need to create those resources.  First in the `CoreStore.Web` project root, I'll add a new folder named `Controllers` to follow the ASP.NET Core MVC convention.  Inside of the folder, I'll continue to honor the convention with a C# file named `HomeController.cs`.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure02.png)

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

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure03.png)

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

This last step is curious.  Currently, Visual Studio Code does not provide Intellisense for referenced projects unless the editor is restarted. I'm not sure what causes this, but fortunately the solution is simple.  Just open the Command Palette with Cmd-Shift-P on macOS or Ctrl-Shift-P on Windows.  In the search bar, type `reload` and select the `Reload Window` option.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure04.png)

Figure 4. Reloading the editor

# The First Test

The `dotnet` CLI added a basic test file when the project was created.  It's in the `UnitTest1.cs` file.  I'll start off by renaming this file to `ControllerTests.cs`.  Then I'll open `ControllerTests.cs` and rename the `UnitTest1` class to `ControllerTests`.  

The class itself does not need to be marked explicitly as containing unit tests; the tests themselves are marked with a `[Fact]` attribute.  The `dotnet` CLI has created one of these for me.  I'll rename the test to something more descriptive, such as `VerifyIndexViewType`.  The `ControllerTests` class now looks like this:

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

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure05.png)

Figure 5. Errors in the editor

This will generate a new a red lines which mean errors.  So I'm going to fix those next.  The first one, under `HomeController`, is easy to correct. Putting the cursor on the `HomeController` text and pressing <kbd>Cmd</kbd>+<kbd>-</kbd> on macOS or <kbd>Ctrl</kbd>+<kbd>-</kbd> on Windows will bring up the Quick Fix popup menu.  This will give the option to add the `CoreStore.Web.Controllers` namespace to the file.  In this case, fixing one error causes another.  This one is a little more subtle.  Hovering over the error will show the following.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure06.png)

Figure 6. The missing assembly

This error is complaining about the `IActionResult` type returned by the `Index` action.  `IActionResult` is in the assemblies for ASP.NET Core MVC which are currently not added to the test project.  In ASP.NET Core 2.0, all of the assemblies for ASP.NET Core are in a single NuGet package: `Microsoft.AspNetCore.All`.  I'll add that to the test project with the `dotnet` CLI.  Be sure to run this command from the test project folder in the integrated terminal.

```
> dotnet add package Microsoft.AspNetCore.All
```

This will trigger a prompt to restore the packages in the project as the new one has been added to the test project .csproj.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure07.png)

Figure 7. The restore prompt

Click `Restore` to permit this and restore the packages.  To get rid of the red line, the editor may need to be reloaded again.  The last error is for the `ViewResult` type in the `Assert` call.  It can also be resolved with the Quick Fix.

And that's the code for the first test.  So what does it mean?  Let's look at the three parts of a unit test and how they relate to `VerifyIndexViewType`.

* Arrange - This is where the prequisites for the test are created.  In this case all that is needed is an instance of the `HomeController`.
* Act - This is the action that is being tested by the unit test.  Here the action is the `Index` method.
* Assert - Here is where the result of the Act step is verified. The assertion API is provided by xUnit.net. The `Assert` class has many methods that do a variety of checks. The `IsType` method takes a generic type and then compares that to the type of the method parameter. Here, it's checking to make sure that the type of the `result` is identical to `ViewResult`.

Notice that Visual Studio Code has added some links above the signature of the test method.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure08.png)

Figure 8. Test method links

Clicking on the `run test` link will build the `CoreStore.Web` project and the `CoreStore.Tests` project, look for and execute unit tests in the test project, and finish by reporting the results.  

And for this first run, my very simple test passed.  

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure09.png)

Figure 9. All tests passed

Just so we can see what a failing test looks like I'll change the generic type in the `Assert` call to `string`.

```
Assert.IsType<string>(result);
```

Clicking the `run tests` link again will build and run the tests, but the test failed this time around.  As the summary states, `result` was expected to be of type `System.String`, but the actual type was `Microsoft.AspNetCore.Mvc.ViewResult` resulting in a failed assertion.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure10.png)

Figure 10. A failing test

# Adding the Model

Let's go deeper into the result of an action method.  The `ViewResult` has a `Model` property referring to the model object passed to the `View` method in the action method. Right now, none of my action methods do this so I'll add a new one to the controller class. But first, I need a model class.  So I'll add a `Models` folder to the `CoreStore.Web` project and then a new `Product.cs` file for the model object.

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

In `ControllerTests.cs` (in `CoreStore.Tests`) add a new test method. Use the Quick Fix to resolve any namespaces that haven't been included.

```
[Fact]
public void VerifyListActionProductCount()
{
    var controller = new HomeController();
    var result = Assert.IsType<ViewResult>(controller.List());
    var model = Assert.IsType<List<Product>>(result.Model);
    Assert.Equal(2, model.Count());
}
```

There are several interesting things going on here.  First is the call to `Assert.IsType` in the second line. This time I am capturing the return value. With this method, if the assertion passes, it will return the parameter cast to the type parameter. So `result` will be a `ViewResult` instead of an `IActionResult`. This is nice because in this test I want to count the number of `Product` in the model.  `ViewResult` has a `Model` property, `IActionResult` does not. The `Model` property is of type `object` but I need it to be a collection of `Product` to count them. So I rely upon `Assert.IsType` again.  With `model` as an `List<Product>` I can make the assertion. This time I am using `Assert.Equal` which compares an expected and actual value. This method has several overloads with different types.

Running the test with the `run test` link produces this output.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure11.png)

Figure 11. Running the model test

# Adding a repository

To simplify data access, the concept of a repository class is often used.  This is an abstraction that talks to code which actually does the heavy lifting of talking to a database, such as a context class in Entity Framework Core.  

But these data access implementations can be swapped out.  One might make the move from SQL Server to Azure Document DB.  To facilitate this, the repository defines an interface and two implementations of the interface, one for SQL Server and Document DB, are created. This approach also helps with testing as we will soon see.  

But first I'll set up the repository. In a new `Core` folder I'll add a file to contain the interface `IProductRepository.cs`.

```
namespace CoreStore.Web.Core
{
    public interface IProductRepository
    {
        List<Product> ListProducts();
        Product GetProductById(int id);
    }
}
```

Obviously a real world repository would be more extensive, but this will do for our purposes. The repository implementation will also be equally simplistic.  I'll put that in another file `MemoryProductRepository.cs` in a new `Infrastrucure` folder.

```
using System.Collections.Generic;
using System.Linq;
using CoreStore.Web.Core;
using CoreStore.Web.Models;

namespace CoreStore.Web.Infrastrucure
{
    public class MemoryProductRepository : IProductRepository
    {
        private static List<Product> products = new List<Product> {
            new Product { ID = 1, Name = "Apples", Price = 1.50m },
            new Product { ID = 2, Name = "Bananas", Price = 2.00m }
        };
        public Product GetProductById(int id)
        {
            return products.Where(p => p.ID == id).FirstOrDefault();
        }

        public List<Product> ListProducts()
        {
            return products;
        }
    }
}
```

The reason the class is named `MemoryProductRepository` is because all of the products are stored in memory.  This class does not talk to a database.  And this is just to keep things simple.  For now, pretend that the repository implementation does talk to a database.  The controller will make use of the repository.  In the `HomeController` class add a `private` instance of the `IProductRepository` and then add a constructor to initialize it.

```
private IProductRepository productRepository;

public HomeController(IProductRepository _pr)
{
    productRepository = _pr;
}
```

### Dependency injection briefly

If you're not familiar with dependency injection you might be asking how to get the parameter to the `HomeController` constructor as the application code never explicitly calls the constructor. This is where dependency injection comes in. To use dependency injection, you register an interface and a corresponding implementation, IProductRepository and MemoryProductRepository in this case, with a dependency injection container. Then when an instance of the interface is asked for, such as in the constructor parameter, an instance of the implementation will be provided.  Registering the interface and implementation is done in the `Startup` class in the `ConfigureServices` method.

```
services.AddScoped<IProductRepository, MemoryProductRepository>();
```

A detailed discussion of dependency injection is outside the scope of this guide.  But I wanted to reveal some of the magic to avoid confusion.

### Finishing the controller

The action methods will now make use of the repository.

```
public IActionResult List()
{
    return View(productRepository.ListProducts());
}

public IActionResult Details(int id)
{
    var product = productRepository.GetProductById(id);
    if (product == null)
    {
        return View("NotFound");
    }
    else
    {
        return View(product);
    }
}
```

I also need to add two new views, one for the `Details` action method:

```
@model CoreStore.Web.Models.Product

<ul>
    <li>@Model.Name</l1>
    <li>$@Model.Price.ToString("F2")
</ul>
```

And a `NotFound` view that I'll add to the `Views/Shared` folder because it's not specific to the `HomeController`.

```
<h1>The requested resource was not found, sorry!</h1>
```

The `CoreStore.Web` project structure now looks like:

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure12.png)

Figure 12. `CoreStore.Web` project structure 

# Mocking

The last test I'm going to write will test the `Details` action method.  But opening the `ControllerTests.cs` file shows that the existing tests are broken.  The `HomeController` constructor now expects a parameter which should be an implementation of `IProductRepository`.  I'll add a new instance to each test.

```
var productRepository = new MemoryProductRepository();
var controller = new HomeController(productRepository);
```

Now I'll verify that the tests still pass by running `dotnet test` from the command line in the `CoreStore.Tests` project folder to run all of the tests at once.

```
> dotnet test
```

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure13.png)

Figure 13. All tests still passing

But now I have a problem.  Remember that I said to pretend the `MemoryProductRepository` actually **does** talk to a database?  What if the connection to that database failed?  Or the wrong connection string was used?  My tests would fail but it would not be the fault of the web app.  So at this point I have changed from unit tests to integration tests.  What I need, is a way to ensure that a call to the API exposed by the repository will always succeed.  That way I can be sure that any errors are isolated to the web app and didn't happen somehwere else in the code.

### Meet Moq

[Moq](https://github.com/moq/moq4) (pronounced 'mock' or 'mock-u') is a mocking library.  It will create mock implementations of interfaces that will always behave the same way according to predefined conditions setup with a fluent API.  Moq is also distributed as a NuGet package so I'll add it to the test project with the `dotnet` CLI.

```
dotnet add package Moq
```

Visual Studio Code will need to restore the packages.

Now I'll create a mock implementation of `IProductRepository` in the `VerifyIndexViewType` test method.  This will replace the `MemoryProductRepository` object.

```
var productRepository = new Mock<IProductRepository>();
var controller = new HomeController(productRepository.Object);
```

This mock does absolutely nothing except satisfy the requirement of the constructor parameter.  And that's another thing, mock objects should be light-weight to avoid slowing down the unit tests.  Now I can click the `run test` link to make sure this test is passing.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure14.png)

Figure 14.  Running a test with a mock repository

For the next test method, I will need to do a little extra setup on the mock repository.

```
var productRepository = new Mock<IProductRepository>();
productRepository.Setup(x => x.ListProducts()).Returns(new List<Product>{
    new Product(), new Product(), new Product()
});
var controller = new HomeController(productRepository.Object);
```

Here the `Setup` method is saying that when the `ListProducts` method is called, return a new `List` with 3 `Product` objects.  Notice that the `Product` objects are empty. I didn't fill in any of the properties. That's because for this test, the property values are irrelevant. I'm only worried about having the correct count. Also, I'll need to update the `Assert.Equal` call expected value to 3.

```
Assert.Equal(3, model.Count());
```

Clicking on the `run test` links shows that this test is still passing.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure15.png)

Figure 15. Testing the mock repository

Now I want to write a new test for the `Details` action method. The `Details` action method is unique because it takes a parameter. Here is how to handle that test.

```
[Fact]
public void VerifyDetailsActionReturnsProduct()
{
    var productRepository = new Mock<IProductRepository>();
    productRepository.Setup(x => x.GetProductById(It.IsAny<int>())).Returns(new Product {
        ID = 1, Name = "Apricots", Price = 1.00m
    });
    var controller = new HomeController(productRepository.Object);
    var result = Assert.IsType<ViewResult>(controller.Details(1));
    var model = Assert.IsType<Product>(result.Model);
    Assert.Equal("Apricots", model.Name);
}
```

The main difference in this test from the previous one is the `It.IsAny<int>` call in the `Setup` method. This tells Moq to setup the mock repository to accept any value to the method, as long as it is of type `int`.  Then it will return a hardcoded `Product` which will be in the `Model` that can be examined in the `Assert.Equal` call.  Running all of the tests now with `dotnet test` shows they are all passing again.

![description](https://raw.githubusercontent.com/douglasstarnes/CoreStoreimages/master/Figure16.png)

Figure 16. All tests are passing

# Summary

This guide has only scratched the surface of xUnit.net, Moq and testing in general. The tests that I wrote here were simple for the sake of brevity. What it did accomplish was to touch as many parts of the testing workflow as possible in a short amount of time. Now you have a foundation to build on when starting to test your own applications.

___
I hope you found this guide informative and interesting. If you have any comments or questions, please leave them in the discussion section below! Thanks for reading!