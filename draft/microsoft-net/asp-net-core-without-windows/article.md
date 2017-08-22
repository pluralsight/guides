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


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Figure 15 - Searching for Program.cs**

I'm not going to labor over this file too much other than to say that 
