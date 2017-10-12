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

Storing and retrieving session data is quite simple.  I'll start off by looking at sessions from within a controller class in an MVC application.  Later in the guide I'll show how to access the session from other parts of the application.  Inside of the on `HomeController` class I'll make a new action method, `SetSessionData`:

```
public string SetSessionData()
{
    var random = new Random((int)DateTime.Now.Ticks);
    var randomInt = random.Next(1000000);
    HttpContext.Session.SetString("randomInt", randomInt.ToString());
    var message = $"The session value has been set to {randomInt}";
    return message
}
```

In a later demo I'll show off something more interesting but right now this will do to get started.  The first two lines of the method generate a random integer with a maximum value of 1,000,000.  The third line is where the value is stored in the `Session`.  The `Session` is retrieved from the `HttpContext` and then a `string` is set with a key and value.  Now it might seem kind of restrictive that the `Session` only allows `string` content but I'll show a way around this limitation soon.  After storing the random number it is sent back to the client.  Don't forget what is it because the next action method retrieves it.

```
public string GetSessionData()
{
    var randomInt = HttpContent.Session.GetString("randomInt");
    if (randomInt == null)
    {
        return "The session value has not been set";
    }
    else
    {
        return $"The session value is {randomInt}";
    }
}
```

Here the first line again gets the `Session` from the `HttpContext`.  This time instead of setting a `string`, it retrieves a stored `string` by key with the `GetString` method.  What if no value exists for the key in the `Session`?  The result will be `null` which is why I have to do the check.  The next three images show the result of running the application.

Figure 1. No session data

In the first image, the session value has not been set for the `randomInt` key.  So the value returned is `null`.

Figure 2. Setting the session data

The second image shows that the `randomInt` key has been set to 442568.

Figure 3. Retrieving the session data

The third image shows the the retrieved value is still 442568, thus confirming that the session code works.

# More than strings

Of course the world does not store everything as a string.  Or does it?