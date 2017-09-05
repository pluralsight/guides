# Introduction

The web, more specifically HTTP, is stateless.  Any data that was created or changes made to existing data are gone at the end of the HTTP request lifecycle.  There are many persistence methods to accomodate this shortcoming.  Data can obviously be stored in a database.  Cookies are another option that keeps data on the client.  However, cookies are insecure and rely upon the user's participation to make sure that data is not hijacked.  So they are not the ideal solution either.  This guide discusses sessions, a way to maintain state between requests that is managed on the server.  

> Sessions do use cookies.  However, they store only an anonymous session ID which would be useless if acquired by another application.

I'll be discussing sessions within the context of ASP.NET Core.  ASP.NET Core is cross platform and to prove this I'll be using the `dotnet` CLI tool installed with the .NET Core SDK and VIsual Studio Code with the C# extension installed.  The concepts and code however, are identical across platforms.  You could just as easily implement this on Visual Studio for Windows or on Visual Studio for Mac.

# Setup

To keep things simple, I'll use the empty ASP.NET application template that comes with the `dotnet` CLI tool.  This template is named `web` and I will create the application with this command:

```
> dotnet new web -o SessionDemo
```

This template does not provide support for ASP.NET Core MVC but adding it is not a big deal.  In the `Startup.cs` file in the root of the project, find the `ConfigureServices` method.  Add the following line:

```
services.AddMvc();
```

Then in the `Configure` method, remove the call to `app.Run` and replace it with this code:

```
app.UseMvcWithDefaultRoute();
```

This will have the same effect as defining a default route with the format:

```
{controller=Home}/{action=Index}/{id?}
```

Now the controller and view to prevent a 404 need to be added.  I won't discuss that here in much detail.  For now all that is needed is a `HomeController` class that has an `Index` action method which returns a default view.  Also, add a view for the `Index` action method which contains some HTML just so that the application will run.

# Adding Session Support

Getting sessions into the application is about the same amount of work as adding MVC support.  This takes place in the `Startup.cs` file in the same two methods.  First in the `ConfigureServices` method add this line **before** the call to `AddMvc`.

```
services.AddSession();
```

Again, the session support must be added before the MVC support.

Then in `Configure`, again before using MVC, call:

```
app.UseSession();
```

And that's it!  Sessions are now enabled for the application.

# Accessing Session Data

