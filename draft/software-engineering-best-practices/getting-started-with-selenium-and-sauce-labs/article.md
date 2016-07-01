As developers, we are used to unit test model classes or other components, but sometimes, some functionality must be tested at the user interface level, especially in web applications. Selenium is a tool that automates this kind of testing. 

In this tutorial, you'll learn how to create a Selenium test through the following steps:
- First, by using the [Selenium IDE](http://www.seleniumhq.org/projects/ide/), a Firefox plugin to record actions on the browser so they can be replayed later.
- Then, you'll turn that test into Java code and redesign it using the Page Object pattern. 
- Finally, you'll modify the test so it can run in the cloud using [Sauce Labs](https://saucelabs.com/).

As mentioned, we'll use Java to code the tests, but the tools (like Selenium IDE and Sauce Labs) and concepts (like the Page Object pattern) apply to all the other languages supported by Selenium (Ruby, C#, Python, Node.js, JavaScript, and PHP).

The source code of all the tests (is available on Github](https://github.com/eh3rrera/selenium-saucelabs-example).

# Requirements

### Java Environment
You must have [JDK 1.6 or higher installed](http://www.oracle.com/technetwork/es/java/javase/downloads/index.html).

We'll use [Eclipse](https://eclipse.org/downloads/) and [Maven](https://maven.apache.org/install.html), so you should have these installed also.


### Selenium IDE
We'll need the Selenium IDE to record/play tests on the browser. It is available as a Firefox plugin, so make sure you have this browser installed.

Go to http://docs.seleniumhq.org/download/ and scroll down the page until you find the Selenium IDE section:
![Selenium IDE Download](https://raw.githubusercontent.com/pluralsight/guides/master/images/624f278c-83c1-4fef-9105-88314c883125.png)

Click the link to download the plugin and install it:
![Install plugin](https://raw.githubusercontent.com/pluralsight/guides/master/images/4a1602ff-dd6e-48dc-b93f-0ca7d6f1edef.png)


Upon restarting the browser, you can start the Selenium IDE with this toolbar button:
![Selenium toolbar button](https://raw.githubusercontent.com/pluralsight/guides/master/images/c6819eec-f50f-43bf-8f08-849a0581fa31.png)


Or from the *Tools* menu:
![Selenium menu option](https://raw.githubusercontent.com/pluralsight/guides/master/images/bfc13ebd-b3c2-4955-853b-cd7d67902f69.png)

Open the Selenium IDE with either option to check it was installed properly:
![Selenium IDE Window](https://raw.githubusercontent.com/pluralsight/guides/master/images/72fa0c75-6897-4162-871e-5938691fab5d.png)


### Sauce Labs
We'll need Sauce Labs to run the test in the cloud.

