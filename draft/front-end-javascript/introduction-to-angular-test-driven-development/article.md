1. npm install angular
npm install angular --save
2.Install Karma
npm install -g karma --save-dev
3.Install Jasmine
npm install karma-jasmine jasmine-core --save-dev
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