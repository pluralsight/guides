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
 #### Installing packages
 
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

#### Configuring Karma
 Configuring your testing environment is done through a configuration file (<code>karma.conf.js</code>), similarly to configuring <code> package.json </code>, which we used to configure the project environment.
 
There is a [great number of options](http://karma-runner.github.io/1.0/config/configuration-file.html) for configuring Karma. The most essential are the following:

- **Frameworks** - the testing frameworks that are going to be used. In this guide, we are using jasmine.
- **Files** - files that will be used for the tests. You would normally include both the framework files (in this case, Angular) as well as the project and the testing files themselves.
- **Browsers** - You can specify which browsers must Karma use in order to run its tests.

Before starting, create two folders in  your working directory.

```
mkdir app 
mkdir tests 
```
We'll use <code> app </code> to store our applicaton code and <code>tests</code> to keep our tests. This way, we'll keep the two separated. In more advanced cases you would opt for a more generic approach by using wildcards and storing the test and application files together. 

With your terminal in the directory of the project, run:

```bash
karma init
```
Answer the questsions as following:
- Select Jasmine as your testing framework.
- Select 'Chrome' as your browser.
- Specify the paths to your js and spec files: <code> 'app/\**.js', 'test/*\*.js' </code>
- Specify the path to your Angular and ngMocks files: <code>'node_modules/angular/angular.js','node_modules/angular-mocks/angular-mocks.js'</code>


In the end, you should end up with a <code>karma.conf.js</code> file looking like this: 
```javascript
module.exports = function(config) {
  config.set({
    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: '',
    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine'],
    // list of files / patterns to load in the browser
    files: [
      'node_modules/angular/angular.js',
      'node_modules/angular-mocks/angular-mocks.js',
      'app/app.js',  //use wildcards in real apps
      'tests/tests.js' //use wildcards in real apps
    ],
    // list of files to exclude
    exclude: [
    ],
    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
    },
    // test results reporter to use
    // possible values: 'dots', 'progress'
    // available reporters: https://npmjs.org/browse/keyword/karma-reporter
    reporters: ['progress'],
    // web server port
    port: 9876,
    // enable / disable colors in the output (reporters and logs)
    colors: true,
    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,
    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: true,
    // start these browsers
    // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
    browsers: ['Chrome'],
    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false
  })
}
```
Once you are done with this step, you are ready to dive into testing features.

## Writing your first test

Writing tests revolves around events and their expected outcomes. When you start writing the tests for your applicaiton, you must always think in terms of desired outcomes your app components wants to produce. 

### Building blocks
Although the concepts are specific for Jasmine, they share many similarities with other behaviour-driven testing frameworks:

- **Suites** - Are the top-level element of the testing framework. They accept a title (explanation) and a function containing one or more specifications. 

<code>describe(string, function)</code>
- **Specs** -  Are constructs that take a title and a function containing one or more expectations. Specs are nested into suites.

<code>it(string, function)</code>
- **Expectations** — are assertions that evaluate to true or false.

<code>expect(actual).toBe(expected)</code> 
- **Matchers** - Are redefined helpers for common assertions. They are the constructs that do the evaluations and decide whether a test has failed or not.
- 
<code>toEqual(expected)</code>

To give you a better idea of what matchers can do, here is a list of the most essential matchers:

```javascript
expect(fn).toThrow(e);
expect(instance).toBe(instance);
expect(mixed).toBeDefined();
expect(mixed).toBeFalsy();
expect(number).toBeGreaterThan(number);
expect(number).toBeLessThan(number);
expect(mixed).toBeNull();
expect(mixed).toBeTruthy();
expect(mixed).toBeUndefined();
expect(array).toContain(member);
expect(string).toContain(substring);
expect(mixed).toEqual(mixed);
expect(mixed).toMatch(pattern);

```
[Here is the full list of matchers.](https://github.com/JamieMason/Jasmine-Matchers)

#### Teardown

Teardowns are another building blocks of tests that is used to "prepare" the code for its specs in a particular suite. Suppose that before or after each spec (i.e a <code>describe</code> function) to do certain setups so that you can test a particular functionality. Instead of doing it for every spec, you can write a <code> beforeEach </code> or an <code> afterEach </code> function in your sute to do that.

```javascript
// single line
beforeEach(module('itemsApp'));

// multiple lines
beforeEach(function(){
  module('itemsApp');
  //...
});
```
 In the block above you can see two ways in which you can initiallize the Angular module in a testing suite using teardown.
 
 #### Injecting
 
 Remember that we installed Angular mocks in the beginning of the guide? 
 
 // Using _serviceProvider_ notation
var $q;
beforeEach(inject(function (_$q_) {
    $q = _$q_;
}));

// Using $injector
var $q;
beforeEach(inject(function ($injector) {
    $q = $injector.get('$q');
}));

// Using an alias Eg: $$q, q, _q
var $$q;
beforeEach(inject(function ($q) {
    $$q = $q;
}));
 
 


