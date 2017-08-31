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
dotnet new web -o CoreStore.Web
```

The test project required a choice.  Two testing frameworks are support by the `dotnet` CLI.  The first is MSTest, which is from Microsoft.  It's a fine product and if you have previous experience with testing Microsoft software in Visual Studio you've likely used it.  But now we are in a cross-platform world and while MSTest with .NET Core is cross platform, there are other options.  One of these is xUnit.NET.  This is an open source project that is inspired by the xUnit concept.  There are many frameworks