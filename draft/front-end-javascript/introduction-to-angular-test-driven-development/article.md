If you have ever taken the step to build something more complex than a simple to-do app with a AngularJS or any other technology, you know that you always end up fixing old things along with the implementation new features. This is a common problem with many technolgies. No matter the language your software is  built with, its architecture and ways of separating concerns, you will sooner or later end up breaking things that previously worked when releasing new builds. This is where [Test-Driven Development](http://hackguides.org) comes in play. 

Unit testing helps you answer all the *'Is this going to work if I insert X?'* or *Does X still return the same results now that I have implemented Y?'* scnearios. It does it by simulating actions with mock data and checking if everything is outputted in the format you expect it to be.

Angular is built with [testing in mind](https://docs.angularjs.org/guide/unit-testing). If you are new to writing unit tests, starting off with Angular the starting point to get deeper into the topic. The framework allows simulation of server-side requests and abstraction of the [document object model](https://en.wikipedia.org/wiki/Document_Object_Model) , thus providing an environment for testing out numerous scenarios. Additionally, Angular's dependency injection allows every component to be mocked and tested in different scopes.

In this tutorial, we are going to get through through the gist of test-driven development in Angular. We'll start off by setting up the environment and looking at how [Jasmine](http://jasmine.github.io/2.4/introduction.html), [Karma](https://karma-runner.github.io/1.0/index.html) and [Angular Mocks](https://docs.angularjs.org/api/ngMock) work together to provide an easy and seamless testing experience in Angular.


## Getting started

Every great developer knows what his tools are for. Knowing the role of each of your tools for testing is essential before diving right into writing tests. 

### What does what?

** Karma**  an environment which runs the tests of your applicaiton. Simply put, it's your testing server. Originally started as a [university thesis](https://github.com/karma-runner/karma/raw/master/thesis.pdf), Karma aims to make a framework agnostic environment that automates the running of your unit tests. In its core, it is a [Node](https://nodejs.org/en/) server that watches for changes in your testing and application files, and when such changes occur, it runs them into a browser and checks for mistakes.

** Jasmine ** is an unit testing framework for JavaScript. It is the most popular framework for testing JavaScript applications, mostly because it is quite simple to start with and flexible enough to cover a wide range of scnearios

** Angular mocks ** is an Angular module that is used to mock compontents that already exist in the applicaiton. Its role is to inject various components of your Angular applicaiton (controllers, services, factories, directives, filters) and make them available for unit tests. It can be said that Angular Mocks is the middleman between the Angular components in your applicaiton and the unit testing environment.

### Setting up the testing environment

 To ensure a speedy and efficient setup, I recommend using the [Node Package Manager (npm)](https://www.npmjs.com/) to maintain the dependencies of your Angular project. 
 
 First, let's install the packages we'll need to run the tests globally (done by appending <code> -g </code>) . 
Open your terminal and run the following commands:
 ```
  npm install -g karma
  npm install -g jasmine-core 
```
Then, create a directory where you'll store your project files.
 
 ```bash
 mkdir myitemsapp
 cd myitemsapp
 ```
 Once you are int he directory, start setting up your project dependencies. 
 First, initialize your <code>package.json</code> file:

```bash
 npm init
 ```
 You will get asked several questions regarding your project's details. You can skip them and simply copy the contents below in your <code> package.json </code> file.
 
```json
 {
  "name": "myitemsapp",
  "version": "1.0.0",
  "description": "",
  "main": "karma.conf.js",
  "directories": {
    "test": "tests"
  },
  "dependencies": {
    "angular": "^1.5.7",
    "save": "^2.3.0"
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```
 Then start installing your project's dependencies, starting off with Angular:
```bash
npm install angular --save
npm install karma --save-dev
npm install karma-jasmine jasmine-core --save-dev
npm install angular-mocks --save-dev

```
Next, you have to choose a browser launcher for Karma to use:
You can use one for [Chrome](npm install karma-chrome-launcher --save-dev) , [Firefox](https://github.com/karma-runner/karma-firefox-launcher) and [PhantomJS](https://github.com/karma-runner/karma-phantomjs-launcher).
 I will go with Chrome:
 
```
npm install karma-chrome-launcher --save-dev
```
Create two folders in  your working directory.

mkdir app //your script files, controllers,filters etc.
mkdir tests //here we will keep our tests.

 karma.conf.js
karma init
Select Jasmine as your testing framework.
Select browser

Specify the paths to your js and spec files. Eg. 'app/*.js', 'test/*.js‘.

Open up your karama.conf.js and add the location of angular.js in to the files array.
node_modules/angular/angular.js
Add the location for ngMock just below that.
node_modules/angular-mocks/angular-mocks.js