# Introduction

When .NET Core was announced at Visual Studio Connect() in 2014, one of the most exciting features was that it would be supported across multiple platforms namely Windows, macOS and various distributions of Linux.  However, this also introducted a challenge, which was tooling.  The preferred tool for writing ASP.NET applications at the time was Visual Studio, running on Windows, deploying to Windows.  What would the tooling and developer experience look like in the abscence of Windows?

At the annual Microsoft Build developer conference, six months later in April 2015, Microsoft introduced the world to Visual Studio Code.  Inspired by Visual Studio for Windows, Visual Studio Code is a lightweight text editor which is both free and cross-platform with builds for Windows, macOS and Linux which are conveniently the same platforms targeted by .NET Core.  Built on the Electron shell from Github and written in TypeScript, Visual Studio Code is not even a .NET application, even on Windows.  Eventually, Visual Studio Code was made open source on Github.  

With the addition of support for extension authoring, the capabilities of Visual Studio Code began to increase very quickly.  Today, with the help of extensions, Visual Studio Code is a major player and worthy competitor to other established editors including Visual Studio, Sublime Text and Atom.  This guide will look at what is needed to setup Visual Studio Code, along with the .NET Core SDK, on any of the three supported platforms to create a database-driven web application with a REST-ful API.

# Installation

Let's look at how to set up the development environment.  In addition to the .NET Core SDK and Visual Studio Code, you'll need Docker and an Azure account to follow through the complete deployment.  But only .NET Core and Visual Studio Code will be needed to actually build the application locally.

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
