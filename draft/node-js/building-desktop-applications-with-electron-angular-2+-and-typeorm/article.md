
Throughout this tutorial, we are going to learn how to integrate TypeORM (a TypeScript ORM) with Electron platform to build data extensive cross-platform desktop applications. In the same time we are going to build an example application for products inventory management to show different concepts related to either TypeORM or Electron.

Before getting started with building our example Electron app, lets first talk a little bit about the tools we are using.

## What is Electron?

Electron is a project, created by Github, for building cross-platform desktop applications using web technologies such as JavaScript/TypeScript, HTML and CSS. Electron is based on Chromuim browser and Node.js platform so think about that? you have the whole Node.js modules, available from NPM, and an embedded browser to make use of the latest client side frameworks, libraries and browser APIs such as Angular and React etc. To build your desktop application. It deosn't end here Electron offers you also a rich API to integrate with the underlying operating system such as Tray icons, dialogs and menus etc.

Thanks to Electron, web developers can use their existing skills to build applications for Linux, MAC and Windows without having to learn traditional languages such as Java, C++ or C#.

## What is TypeORM?

TypeORM is an Object-Relational Mapper based on TypeScript, it simply lets you work with SQL based/relational databases without writing SQL but TypeScript, a superset JavaScript created by Microsoft which adds strong types and classical OOP concepts to JavaScript.

TypeORM is similar to ORMs such as Java Hibernate, .NET Entity framework and Doctrine which are known to be the most popular and powerful ORMs in the world.

## What is Angular 2?

Angular 2 is the next version of the popular client side framework, AngularJS built by Google, it was completely rewritten from scratch using TypeScript instead of plain JavaScript. Angular 2 or just Angular is component based which means your application is a set of root and child components which encrourages separation of concerns and maximum code re-usability.



## Designing a Simple Database For Products Inventory Management 

Before writing actual code, lets first discuss our application database requirements. Our application needs to be able to:

* Persist information about products such as: id, name, upc, family, location, price and quantity.
* Persist information about product locations: id, name.
* Persist information about product families: id, name.
* Persist information about transactions: id, price, quantity and product id.

Each product belongs to a family and a location.
Each transaction is related to some product. 





