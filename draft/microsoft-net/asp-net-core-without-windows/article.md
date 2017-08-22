#Introduction

When .NET Core was announced at Visual Studio Connect() in 2014, one of the most exciting features was that it would be supported across multiple platforms namely Windows, macOS and various distributions of Linux.  However, this also introducted a challenge, which was tooling.  The preferred tool for writing ASP.NET applications at the time was Visual Studio, running on Windows, deploying to Windows.  What would the tooling and developer experience look like in the abscence of Windows?

At the annual Microsoft Build developer conference, six months later in April 2015, Microsoft introduced the world to Visual Studio Code.  Inspired by Visual Studio for Windows, Visual Studio Code is a lightweight text editor which is both free and cross-platform with builds for Windows, macOS and Linux which are conveniently the same platforms targeted by .NET Core.  Built on the Electron shell from Github and written in TypeScript, Visual Studio Code is not even a .NET application, even on Windows.  Eventually, Visual Studio Code was made open source on Github.  

With the addition of support for extension authoring, the capabilities of Visual Studio Code began to increase very quickly.  Today, with the help of extensions, Visual Studio Code is a major player and worthy competitor to other established editors including Visual Studio, Sublime Text and Atom.  This guide will look at what is needed to setup Visual Studio Code, along with the .NET Core SDK, on any of the three supported platforms to create a database-driven web application with a REST-ful API.

