In this tutorial, we're going to build a chat using Pusher's presence channels with the functionality to request e-signed Non-Disclosure Agreements (NDAs) to its members using the HelloSign API.

The stack will be the following:
- Java 7 or higher
- Maven as the build manager
- Spring Boot as the server-side framework
- H2 as in-memory database
- Thymeleaf as the server-side template engine
- jQuery and Handlebars for the client-side interaction

We're going to use Pusher and HelloSign webhooks to receive events from these APIs and we'll use ngrok to keep everything in a local environment.

The app's design is based on this [pen](http://codepen.io/drehimself/pen/KdXwxR) and it works in the following way. First, a user creates a chat:

![Create chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/a4a6062a-a807-42c4-9720-afdff82beb11.gif)


Then, another user joins it:

![Join chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/5d064ba2-a70a-4199-bad6-2a1b3c8701f5.gif)


At any time, the chat owner can send a request to sign an NDA:

![Sending NDA](https://raw.githubusercontent.com/pluralsight/guides/master/images/401eb9e9-e6aa-4468-9446-230a9724454e.gif)


When a member of the chat signs the NDA, a notification is sent with a link to view the signed document:

![Notification signed document](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf249643-5131-47b5-b8fb-09720271e6ad.gif)



The entire source code of the final app is on [Github](https://github.com/).


# Requirements

### Java environment

You'll need JKD 7 at least, however, [JDK 8 is preferred](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

You'll also need [Maven 3.0](https://maven.apache.org/download.cgi) or higher [installed](https://maven.apache.org/install.html).

An IDE, like [Eclipse](https://eclipse.org/downloads/), [IntelliJ](https://www.jetbrains.com/idea/) or [Netbeans](https://netbeans.org/downloads/), will make things easier but it's not required.


### Pusher

Create a free account at https://dashboard.pusher.com/accounts/sign_up.

When you first log in, you'll be asked for some configuration options:

![Pusher Account](https://raw.githubusercontent.com/pluralsight/guides/master/images/528b9445-05ee-456d-bf36-ca8c83c95bd9.png)


Enter a name, and choose *Javascript* as your front-end tech and *Java* as your back-end tech.

Then go to either the *Getting Started* or *App Keys* tab to copy your App ID, Key, and Secret credentials, we'll need them later.


### HelloSign

Sign up at https://www.hellosign.com/. Your free account will have the following limitations:
- Send 3 documents every month for free
- One sender
- No templates

But don't worry, these limitations don't apply in test mode, the mode where we're going to work.

### Ngrok
When a member is added to the chat or an NDA is signed, a webhook will be triggered (think of it as a callback). This means an HTTP request will be made to our server, so we'll need to deploy our application on the cloud or keep it locally and use a service like [ngrok](https://ngrok.com/) to make it publicly available.

Ngrok proxies external requests to your local machine by creating a secure tunnel and giving you a public URL.

ngrok is a Go program, distributed as a single executable file (no additional dependencies). So for now, just download it from [https://ngrok.com/download](https://ngrok.com/download) and unzip the compressed file.

Now that we have all we need, let's create the app.


# Setting up the application

One of the easiest ways to create a Spring Boot app is to use the project generator at [https://start.spring.io/](https://start.spring.io/).

Go to that page and choose to generate a Maven project with the following dependencies:
- Web
- H2
- Thymeleaf
- JPA

Enter a Group ID and an Artifact ID and generate the project:

![Sprint Initializr](https://raw.githubusercontent.com/pluralsight/guides/master/images/17e7968c-3234-4f8f-822d-ef1e34906060.png)


Unzip the content of the downloaded file. At this point, you can import the project to an IDE if you want. For example, in Eclipse, go to *File -> Import* and choose *Existing Maven Projects*:

![Import Maven Project into Eclipse](https://raw.githubusercontent.com/pluralsight/guides/master/images/83970ac2-1e10-4e8d-b5cf-57c53ed8c423.png)

Now let's add some configurations to the `pom.xml` file. In the `properties` section, change the Java version if you're not using Java 8 and add the following line:
```xml
<spring.version>4.3.1.RELEASE</spring.version>
```

The latest version of the Spring Framework at the time of this writing is `4.3.1.RELEASE`. This line will ensure Spring Boot uses this version.

Also, in the `dependencies` section, add the dependencies we'll need for our project:
```xml
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-lang3</artifactId>
	<version>3.4</version>
</dependency>

<dependency>
  <groupId>com.pusher</groupId>
  <artifactId>pusher-http-java</artifactId>
  <version>1.0.0</version>
</dependency>

<dependency>
	<groupId>com.hellosign</groupId>
	<artifactId>hellosign-java-sdk</artifactId>
	<version>3.4.0</version>
</dependency>
```

Inside `src/main/java`, we'll work with the following package structure:
- `com.example.config` will contain configuration classes
- `com.example.constants` will contain interfaces with constants values used in the app
- `com.example.model` will contain the JPA entity models
- `com.example.repository` will contain the Spring JPA interfaces to work with the model
- `com.example.service` will contain the business service classes of the app
- `com.example.web` will contain the Spring MVC controllers of the app
- `com.example.web.vo` will contain the objects used for communication between the view and the controllers

Inside `src/main/resources`, we'll put some configuration files in addition to the following directory structure:
- `static/css` will contain the CSS style files used in the application
- `static/img` will contain the images used in the application
- `static/js` will contain the Javascript files used in the application
- `templates` will contain the Thymeleaf templates used in the application

So let's start by creating the `com/example/web/ChatController` class. Create a new file with the following content:
```java
@Controller
public class ChatController {
	
	@RequestMapping(method=RequestMethod.GET, value="/")
    public ModelAndView index() {
		ModelAndView modelAndView = new ModelAndView();
		
		modelAndView.setViewName("index");
		modelAndView.addObject("text", "Hello World!"); 
        
        return modelAndView;
    }
}
```

This controller defines a `/` route that shows a `index` template. Create the file `src/main/resources/templates/index.html` with some HTML content like:
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>NDA Chat</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="${text}" />
</body>
</html>
```

Using Thymeleaf syntax, this will print the `text` variable defined in the controller. You can learn more about [Thymeleaf here](http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html).

Run the application either by executing the `com.example.NdaChatApplication` class on your IDE or on the command line with:
```
$ mvn spring-boot:run
```

Additionally, on the command line, you can create a JAR file to execute it:
```
$ mvn package -DskipTests
$ java -jar target/nda-chat-0.0.1-SNAPSHOT.jar
```

When you open `http://localhost:8080/` in a browser, you should see something like the following:
![Hello World on localhost](12-hello-world-localhost.png)

Now, let's take a moment to configure ngrok.