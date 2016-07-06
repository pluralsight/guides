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
 
 Ope
1. npm install angular
npm install angular --save
2.Install Karma
npm install -g karma --save-dev
3.Install Jasmine
npm install karma-jasmine jasmine-core --save-dev
npm install jasmine-core -g
4.Install ngMock

npm install angular-mocks --save-dev

ngMock allows you to inject and mock angular services to help you test your application.

- Browsers
Install browser launcher on which you want karma to run your tests. We need to install atleast one browser. I’ll use PhantomJs.
- npm install karma-phantomjs-launcher --save-dev
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