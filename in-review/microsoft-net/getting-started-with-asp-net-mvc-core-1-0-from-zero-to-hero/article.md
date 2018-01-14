## Getting started with all the power of ASP.net MVC Core 1.0

Detail:
This is going to be my first tutorial written in English, so if you find a bug in my spelling system I will appreciate you report it to me!
Well, first we need to accept the terms and conditions in order to continue with this tutorial:

1.  Focus your attention first on this tutorial and after you complete it you will create it by your own.
2.  The tools we need are: Visual Studio Code, so, you can be on your Windows, Mac or in a GNU/Linux Distribution, Visual Studio Code is Cross-platform.
3.  This is just the starting point, promise with yourself to expand, keep this in mind: Today I will better than yesterday and tomorrow better than today.
4.  Say this to yourself: I am going to help more people interested on this topic :)

Ok! perfect! Once you accepted this Terms and Conditions, let's rock!

This is the idea:
--------------------------
* Installing the tools
* Overview about MVC Pattern.
* Create a simple project from scratch (No validation).
* Add MVC support.
* Add CRUD operations.
* Playing with action filters.
* Create a custom helper to increase our productivity.
* Add Sessions.
* Store the data on a database.
* Add Identity : This will take care about users, logins, roles, claims.

## Overview about MVC Pattern.
This is not new, according to the story and thanks to Wikipedia the concept of MVC comes from 1970s... 

## Creating the project
* We need to have ASP.net MVC RC2 Tooling.
* This new framework is very modular, as you advance you install only what you need. This means in few words: high performance because you are going to avoid getting everything to just use a small part.

Let's take a look how does it work:
1. First at all, open Visual Studio CE 2015.
2. File, New Project, We are going to target Net Framework 4.6.1.
3. We are going to start from empty, from nothing, from zero to hero!
4. Now if you are have experience working with ASP.net MVC you will realize there is a new folder structure. Personally I really like it, it makes a lot of sense having package managers for client side and for server side. More organization, better code.

## Add MVC Support
Now we add the support of MVC doing the following steps:
1. Add dependencies to the project.json
2. Go to StartUp.cs and modify it

### Creating our folder structure
1. Create the following folders: Controllers, Models, Views.
2. Under Controllers, we are going to add a Class: HomeController.cs
3. Let's add the three legendary actions: Index, About and Contact. (Seriously, they are legen-wait for it-dary)
4. On StartUp.cs we need to set up 

## Add CRUD operations.
CRUD? If you are a new developer in this world one of the first questions you will have is: What the hell is it? Well, it refers to **C**reate **R**ead **U**pdate and **D**elete Entities.
For make this more interesting and having a clear idea, now let's pick up a topic: How about Sales? Sales is another *Mystic* topic that you will find on Internet in many different languages. Let's start with a couple of classes: Products and Categories and later on we will increment the level.

* What are the properties for a *Product*? Name, Unit Price, Category, and we can continue as long as we require. For this sample we are going to stay with these three, remember this is just the starting point!
* What are the properties for a *Category*? Name, Description. And that's it. We have our basic **classes** ready to continue.

Something we are forgetting is that the machine need an field to distinguish the difference between the records on our table for products.

## Playing with action filters.
First at all, An action filter could be execute... on the request...

## Create a custom helper to increase our productivity.
We have to create over and over the same html, in the following sample we make it simple, instead of copy-pasting lines of code we use a Helper.

## Add Sessions
ViewBags, ViewData, TempData, Sessions...

## Store the data on a database
Here is one of my favorite part: Entity Framework! This ORM was created on the top of ADO.net, it helps to Developer making a clear Data Manipulation using objects (Theory of OOP).

## Add Identity : This will take care about users, logins, roles, claims. 



Final words
------------------------
Thank you for reading this article, if you enjoyed this post I will be happy if you share it! Keep rocking! This is just the beginning. 
