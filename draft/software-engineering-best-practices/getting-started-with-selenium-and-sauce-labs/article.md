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

You can either sign up for a 14-day free trial or for a free account if you want to test an open-source project. For the latter, you just need the repository URL of the project and go to https://saucelabs.com/opensauce/ to fill the details.

Once you have access to Sauce Labs, go to the *My Account* option and look for you Access Key:
![Sauce Labs Access Key](00-sauce-labs-access-key.png)

Save it as we'll need it later.





# The Record and Playback pattern
As its name implies, this pattern is about allowing the users to record their interactions with an application and play them back through a tool later.

This provides an easy start to testing since it doesn't require any experience with a programming language, and it's a fast way to build a test suite.

To show the Record/Playback style of writing test, we're going to use the Selenium IDE and the first lesson of a Markdown tutorial that you can find in http://eherrera.net/markdowntutorial.

To enter the first lesson of the tutorial, go to http://eherrera.net/markdowntutorial/tutorial/emphasis.html:

![Open the first lesson of the tutorial](https://raw.githubusercontent.com/pluralsight/guides/master/images/2b185376-f739-4869-9661-d8cc3e7d4c5f.png)

Now open the Selenium IDE. It will start recording all our actions, so click on the *Start Exercises* button. You'll notice that two commands will be recorded:

![First Recorded Commands](https://raw.githubusercontent.com/pluralsight/guides/master/images/b41ccb36-7d43-480b-9203-adacb8b5c89e.png)

These commands, often called *seleneses*, tell Selenium what to do. For now, just notice how everything you do in the browser is recorded.

Back to the page, it now presents the first exercise. Let's make our test verify this. We have some options to do it, for example:
- We can check that the text *In this exercise, make the words "American Oxygen" bold* is shown.
- We can check that the text box where we enter the solution is shown.
- We can check that the button for revealing the answer is shown.

Think about what is more convenient. The text can change in future revisions of the page, so maybe is not a good choice. The other two options sound like they should always be present at this point of the flow, maybe the text box more than the button for showing the answer (which could disappear under some conditions). For demonstration purposes, let's choose to verify the existence of the button anyway.

If we right-click the *Show me the answer* button and choose the *Show All Available Commands*, the following menu will be shown:

![Right Click Menu to Show Commands](https://raw.githubusercontent.com/pluralsight/guides/master/images/f77e3a8e-a3ca-475f-8920-52c992f06420.png)

The context menu shows some suggested commands with parameters for testing the current element/page. There are three types of Selenium commands:
- Actions are commands that do things like clicking buttons of links. If an Action fails for some reason, the execution of the current test is stopped.
- Accessors examine the state of the application and store the results in variables, like *storeTitle*.
- Assertions check that the state of the application is as expected. There are three types: 
    - *assert* that aborts the test when the assertion fails.
	- *verify* that doesn't abort the test when the assertion fails, it just logs the failure.
	- *waitFor* that waits for some condition to become true (useful for testing Ajax applications). They will succeed immediately if the condition is already true. However, they will fail and halt the test if the condition does not become true within the current timeout setting.
	
So let's choose *verifyElementPresent*. The Selenium IDE window will be updated to show this:

![Record Verification Command](https://raw.githubusercontent.com/pluralsight/guides/master/images/1f2c9c92-c809-4b5d-8585-ab395e3c95ce.png)

However, the test isn't over. Let's do the exercise and make the words "American Oxygen" bold by surrounding them with `**`. Once we do this, the following popup will be shown:

![Input correct answer](https://raw.githubusercontent.com/pluralsight/guides/master/images/d7b8d4c8-8e49-4513-8a5b-851693c301cf.png)

And the Selenium IDE window will be updated to show the command to input the text:

![Selenium IDE window updated](https://raw.githubusercontent.com/pluralsight/guides/master/images/fdd773ce-2354-4d3d-9dfb-9b8fb89aac02.png)

Now let's add another verification. Right-click the popup window and choose *verifyElementPresent //body/div[5]* (Notice that the command was added to the main contextual menu. As you use the IDE, it will try to predict the command you'll want to use):

![Choose command to verify result](https://raw.githubusercontent.com/pluralsight/guides/master/images/847328e8-84ed-4b9f-acba-bfbc3099a705.png)

Again, the Selenium IDE window will be updated:

![Command to verify result](https://raw.githubusercontent.com/pluralsight/guides/master/images/4a54fb47-b774-4b9b-ae84-defc1250a9c3.png)

Notice that, in my case, a *selectWindow* command was added. We can delete this command by selecting it with a right-click and choosing the *Delete* option:

![Delete unwanted command](https://raw.githubusercontent.com/pluralsight/guides/master/images/3489692c-8482-4a83-87e6-018705567235.png)


At this point, the Selenium IDE window should look like this:

![Final script](https://raw.githubusercontent.com/pluralsight/guides/master/images/c9899760-6e59-4e6a-942d-5473e854fcda.png)


Now stop recording and save the test case by going to the *File* menu and choosing the *Save Test Case As...* option:

![Save test case](https://raw.githubusercontent.com/pluralsight/guides/master/images/faeaf3fe-212d-4c67-b1c0-9128257fb6ce.png)


The test case will be saved as an HTML file. This is how the saved file looks like:
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head profile="http://selenium-ide.openqa.org/profiles/test-case">
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<link rel="selenium.base" href="http://eherrera.net/" />
<title>sample-test-case-selenium-ide</title>
</head>
<body>
<table cellpadding="1" cellspacing="1" border="1">
<thead>
<tr><td rowspan="1" colspan="3">sample-test-case-selenium-ide</td></tr>
</thead><tbody>
<tr>
	<td>open</td>
	<td>/markdowntutorial/tutorial/emphasis.html</td>
	<td></td>
</tr>
<tr>
	<td>click</td>
	<td>css=button.button-primary.button-next</td>
	<td></td>
</tr>
<tr>
	<td>verifyElementPresent</td>
	<td>id=btn_answer_1-1</td>
	<td></td>
</tr>
<tr>
	<td>type</td>
	<td>id=editor_1-1</td>
	<td>The music video for Rihanna's song **American Oxygen** depicts various moments from American history, including the inauguration of Barack Obama.</td>
</tr>
<tr>
	<td>verifyElementPresent</td>
	<td>//body/div[5]</td>
	<td></td>
</tr>

</tbody></table>
</body>
</html>
```

To execute the test case, choose the first command (`open`) and click one of the green arrow buttons of the top-left corner of the window (the first will play the entire suite of tests and the second just the current test).

When you do it, you'll see that the test fails:

![Test Error](https://raw.githubusercontent.com/pluralsight/guides/master/images/fdd475d9-df52-4702-8c01-86f447067095.png)

Look the *Log* tab at the bottom of the IDE window and the browser. The problem is that the correct answer popup didn't show up.

Let's analyze how the page works. When the user writes something in the text box, the markdown is generated and compared to the answer to the exercise, which is stored in an array. If there's a match, the popup is shown. Here's the piece of code that does this:
```javascript
// Set up markdown editors
$('.editor').each(function() {
	var elementId = $(this)[0].id;
	var exerciseId = getExerciseId(elementId);
	
	// Set up event to update markdown/html when the user writes something
	$(this).on('keyup', function(event){
		var md = generateMd(exerciseId);
		
		if(exercises[exerciseId] != undefined && md.trim() == exercises[exerciseId]["answer"]) {
			if(exerciseId == "15-1") {
				swal({title:"Excellent job!", text: "You are now a Markdown Master", type: "success"});
			} else {
				swal({title:"Good job!", text: "That's correct. Now you can go to the next exercise.", type: "success"});
			}
		}
	});
	
	// For the initial text
	generateMd(exerciseId);
});
```

As you can see, this happens in the `keyup` event, so we need to fire this event in the test before the verification step.

Return to the Selenium IDE window, right-click the last step, and choose *Insert New Command*:

![Insert-New-Command](https://raw.githubusercontent.com/pluralsight/guides/master/images/4421c091-bc6f-4511-98d3-d4f7ce7fddc4.png)

Below the command table, open the *Command* menu:

![Command Menu](https://raw.githubusercontent.com/pluralsight/guides/master/images/c72094d6-58a7-4873-9d11-406eab94230d.png)

Look at all the commands you have at your disposal. Among them, there's a `fireEvent` command. Select it:

![Fire Event Command](https://raw.githubusercontent.com/pluralsight/guides/master/images/fda44be7-74d3-4b67-bdf7-535a7ef82efe.png)

In the *Target* field, we will input `id=editor-1-1` to reference out text box. We know how to reference this element because of the previous `type` command, but in case we don't know the value of this field, we can click on the *Select* button next to the text box and select and element on the browser to get the value.

In the *Value* field, we will input `keyup`, the event to fire.

Once we have made this change, select the first command (`open`) and run the test again. This time, everything should be fine (also notice how the actions are replayed on the browser):

![Test passed](https://raw.githubusercontent.com/pluralsight/guides/master/images/1880a138-622f-4932-9e6f-43d298cdb742.png)

You can see the value of knowing how the web page works and is structured. 

Save the test and export it as a *Java / JUnit 4 / WebDriver* test case:

![Export Test](https://raw.githubusercontent.com/pluralsight/guides/master/images/e5727186-95ae-4e38-9833-4039936b4e1a.png)

Choose other options to see how the Selenium IDE commands are translated into other languages. Across the supported languages, you can find three versions:
- *WebDriver*, which uses Selenium 2 WebDriver API (this is the preferred option).
- *WebDriver Backed*, which uses Selenium 2 WebDriver to implement the (old) Selenium 1 Remote Control API.
- *Remote Control (RC)*, which uses the (old) Selenium 1 Remote Control API.

If you want to know more about the Selenium IDE [here](http://docs.seleniumhq.org/docs/02_selenium_ide.jsp).





# Understanding Selenium commands

First, let's configure a Java project in Eclipse to run our test class. Go to *File -> New -> Other...* and choose Maven Project:

![Create Maven Project](https://raw.githubusercontent.com/pluralsight/guides/master/images/3d439588-37f8-4473-91dd-ef413d4f90cd.png)

In the next window, check the option *Create a simple project (skip archetype selection)* and click *Next*:

![Create Maven Project](https://raw.githubusercontent.com/pluralsight/guides/master/images/761c2aec-eab7-4f48-a369-2733204a3aff.png)

In the following window, enter the project information, for example:

**Group ID:** com.example

**Artifact ID:** selenium

**Version:** 0.0.1-SNAPSHOT

**Packaging:** jar


![Create Maven Project](https://raw.githubusercontent.com/pluralsight/guides/master/images/5caae031-abc4-4c90-8116-09308e6a6e75.png)

This will create a project with a `pom.xml` file and the following directories:
```
src/main/java
src/main/resources
src/test/java
src/test/resources
```

Open the `pom.xml` file and modify so it looks like this:
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.example</groupId>
	<artifactId>selenium</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.seleniumhq.selenium</groupId>
			<artifactId>selenium-java</artifactId>
			<version>2.53.0</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.saucelabs</groupId>
			<artifactId>sauce_junit</artifactId>
			<version>2.1.21</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
```

We're adding the dependencies for the test framework (JUnit), Selenium and the Sauce Java Helper Library, which will update automatically your Sauce Labs dashboard with information about the tests.

Save the file and if the Maven dependencies are not added automatically to the Eclipse project, right-click the name of the project and choose the option *Maven -> Update Project...*.

Next, copy the exported file of the previous section to the `src/test/java` directory in Eclipse. It should compile with some warnings but without errors:

![Copy class to Eclipse](https://raw.githubusercontent.com/pluralsight/guides/master/images/f14a8172-79c2-4b66-b82f-af23ca36dbbe.png)

Finally, right-click on the class, choose the option *Run As -> JUnit Test*, and see how the test is executed:
