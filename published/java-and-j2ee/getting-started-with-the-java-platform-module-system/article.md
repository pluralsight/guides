# Introduction
Java 9 has introduced the concept of modules to the Java platform. Modules change the way we design and build Java applications, and even though their use is optional, since the JDK itself is now modularized, we must at least know the basics of the Java Platform Module System (JPMS).

In this tutorial, you'll learn what modules are, which problems they solve, how to create them, how they interact with each other, and how we can decouple modules using the service locator pattern. Finally, we'll review how Maven works with the module system.

We'll be using the command line to compile, package, and run the code examples to better understand what's going on. However, you can use the latest (or beta) version of your favorite IDE to do this. Of course, you'll need to have the Java 9 SDK installed. Download it from [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or [here](http://jdk.java.net/9/) and follow the installation instructions.

Let's start by talking about what are modules and what problems they solve.

# Why modules?
In a few words, a module is (generally) a JAR file that contains a set of packages. Just like fields and methods are grouped into classes, and they in turn into packages, packages are grouped into modules.

Modularizing an application means decomposing it into multiple modules that work together.

The important thing is that with modules, you have to explicitly *require* other modules your module depends on, and you have to *export* packages from your module to be used by other modules. This brings some benefits and solves three problems.

First of all, the direct consequence of the module system is that `public` no longer means everyone can access a type. If a class is marked as public, it can be public **only within a module** (if its package is not exported) or only to specific modules that require it.

This promotes stronger encapsulation (the ability to hide parts of the code), which is good because we will no longer have access to internal (private) classes or private members (through reflection) that we are not supposed to use but occasionally change between versions and break our code.

Then, we have the problem of classloaders. Traditionally, the Java Virtual Machine (JVM) has used classloaders to load classes specified in the classpath.

But what happens if you forget to add a JAR or a class to the classpath and run your application?

Maybe nothing, and that's a bad thing. Yes, when the classloader will try to load the missing class, a `ClassNotFoundException` will be thrown, but since classes are lazily loaded, this may not happen at application startup.

And let's not forget all the problems that can happen when we have duplicate JARs or when two libraries require different versions of a third library. This situation even has a name, [JAR (or classpath) hell](https://en.wikipedia.org/wiki/Java_Classloader#JAR_hell).

Projects like [Maven](https://maven.apache.org/) (at compile-time by helping managing dependencies) and [Open Services Gateway initiative (OSGi)](https://en.wikipedia.org/wiki/OSGi) (at runtime with JARs annotated with metadata declaring which packages to export and import from other annotated JARs) had helped us mitigate those classpath problems.

The JPMS doesn't intend to completely replace those projects (we'll talk about Maven in particular in a later section), but Java now has the necessary information to throw not only a compile-time error if a module is not present or visible but also a runtime error when trying to start an application without having access to all the modules it needs or when there's duplicate modules.

The third problem is the deployment size of Java application. Even a simple *Hello World* program requires a full JRE installation with the `rt.jar` file (at around 50 MB) that contains almost all the standard Java classes. 

As said before, in addition to offering a module system for our applications, the JPMS modularizes the JDK itself and, with the help of a tool, [jlink](https://docs.oracle.com/javase/9/tools/jlink.htm), we can create a custom JDK with only the modules we need.

In addition, there's no problem if your application is not ready yet to use modules, the use of modules is optional. Java 9 is backward compatible, if there's no modules information, Java will use the classpath as usual. Besides, an application can even mix a module path and a classpath.

Having explained what a module is and what problems it solves, let's dive into creating our first module.

# My first module

As an example, we are going to use a class with a method that returns a random pre-defined quote related to programming:
```java
import java.util.Random;

public class ProgrammingQuotes {
    private String[] quotes = new String[] {
        "\"To iterate is human, to recurse divine.\"\n" +
                "- L. Peter Deutsch",
        "\"Don't worry if it doesn't work right. If everything did, you'd be out of a job.\"\n" +
                "- Mosher's Law of Software Engineering",
        "\"Good design adds value faster than it adds cost.\"\n" +
                "- Thomas C. Gale",
        "\"Talk is cheap. Show me the code.\"\n" +
                "- Linus Torvalds",
        "\"I don't care if it works on your machine! We are not shipping your machine!\"\n" +
                "- Vidiu Platon",
    };
    private Random rand = new Random();

    private int getRandomIndex() {
        return rand.nextInt(quotes.length);
    }

    public String getQuote() {
        return quotes[getRandomIndex()];
    }

    public static void main(String args[]) {
        System.out.println(new ProgrammingQuotes().getQuote());
    }
}
```

First, we have to put our class in a package because a module cannot use the default package (you cannot export an *unnamed* package).

So let's  use the package `com.example.programming`:
```java
package com.example.programming;
import java.util.Random;

public class ProgrammingQuotes {
  // ...
}
```

Next, we have to choose a name for our module. The recommended approach is to use the same reverse-domain-name pattern that is used for packages. Mark Reinhold, Chief Architect of the Java Platform Group at Oracle, [wrote](http://mail.openjdk.java.net/pipermail/jpms-spec-experts/2017-May/000687.html):
> "Strongly recommend that all modules be named according to the reverse Internet domain-name convention.  A module's name should correspond to the name of its principal exported API package, which should also follow that convention.  If a module does not have such a package, or if for legacy reasons it must have a name that does not correspond to one of its exported packages, then its name should at least start with the reversed form of an Internet domain with which the author is associated."

Of course, you can use any naming convention you want. For example, some people use  short, project-oriented names. 

To make a module, you need to add a `module-info.java` file under the base directory (where your package directories start). By convention, the name of this base directory is the same than the module name. 

This way, the directory structure of our project should be something like this:
```
|─ com.example.programming
|   |─ module-info.java
|   |─ com
|      |─ example
|         |─ programming
|            |─ ProgrammingQuotes.java
```

In the file `module-info.java`, you can do the following:
- Give your module a name
- Require other modules
- Export packages from your module

In this case, it can be as simple as:
```
module com.example.programming {
}
```

Now open a terminal window, make sure to `cd` into the base directory and compile as usual:
```
javac -d out module-info.java com/example/programming/ProgrammingQuotes.java
```

The directory `out` will be created with the compiled `.class` files for `module-info.java` and `ProgrammingQuotes.java`.

Next, package the class into a modular JAR (the only difference with a normal JAR is the presence of the `module-info.class` file):
```
jar cvfe programming-quote.jar com.example.programming.ProgrammingQuotes -C out .
```

This will create the file `programming-quote.jar` with all the classes found in the directory `out` and with `com.example.programming.ProgrammingQuotes` as the main class.

Now run it with:
```
java -jar programming-quote.jar
```

Of course, all this compilation/packaging/running can be done with an IDE, but I wanted to show you that the commands that do these task haven't changed.

However, to run this program as a module, you have to specify the module path (which contains modules, unlike the classpath that contains classes) with the option `--module-path` (or just `-p`) and the main module/class with the option `--module` (or just `-m`), in the format `module/class`:
```
java --module-path programming-quote.jar --module com.example.programming/com.example.programming.ProgrammingQuotes
```

Alternatively, you can use the `out` directory that contains the compile classes:
```
java --module-path out --module com.example.programming/com.example.programming.ProgrammingQuotes
```

However, when your application has more than modules, these options become mandatory on the above commands.

# Working with modules
Let's add a simple graphic interface to our quote generator as another module.

Create another top-level directory `com.example.gui` with the following class:
```java
package com.example.gui;

import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Label;
import javafx.stage.Stage;

public class QuoteFxApp extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception {
        primaryStage.setTitle("Quotes");

        Label label = new Label("A quote");
        Scene scene = new Scene(label, 500, 200);
        primaryStage.setScene(scene);

        primaryStage.show();
    }

    public static void main(String[] args) {
        Application.launch(args);
    }
}
```

And the next `module-info.java` file:
```
module com.example.gui {
}
```

When you execute `javac` to compile the files:
```
javac -d out module-info.java com/example/gui/QuoteFxApp.java
```

A bunch of errors will show up:
```
com\example\gui\QuoteFxApp.java:3: error: package javafx.application is not visible
import javafx.application.Application;
             ^
  (package javafx.application is declared in module javafx.graphics, but module com.example.gui does not read it)
com\example\gui\QuoteFxApp.java:4: error: package javafx.scene is not visible
import javafx.scene.Scene;
             ^
  (package javafx.scene is declared in module javafx.graphics, but module com.example.gui does not read it)
com\example\gui\QuoteFxApp.java:5: error: package javafx.scene.control is not visible
import javafx.scene.control.Label;
                   ^
  (package javafx.scene.control is declared in module javafx.controls, but module com.example.gui does not read it)
com\example\gui\QuoteFxApp.java:6: error: package javafx.stage is not visible
import javafx.stage.Stage;
             ^
  (package javafx.stage is declared in module javafx.graphics, but module com.example.gui does not read it)
com\example\gui\QuoteFxApp.java:10: error: method does not override or implement a method from a supertype
    @Override
    ^
5 errors
```

Apparently, the JavaFX packages are not visible. We need to reference the modules `javafx.graphics` and `javafx.controls`. But why?

The reason behind this is that the JDK has now become modularized, and our application needs to declare which JDK modules it needs.

But why we didn't get any errors in the previous example?

We didn't get any errors because the classes we used were included in the `java.base` module, which is added by default to all modular applications.

Notice that this only happens in modular applications. As mentioned before, Java 9 is backward compatible; if you don't include a `module-info.java` file in your application, everything will work like in previous versions of Java. For example, if you compile the application without `module-info.java`, you'll find that no errors are thrown:
```
javac -d out com/example/gui/QuoteFxApp.java
```

But we are working modules, so we need to *require* the JavaFX modules our application needs in the `module-info.java` file:
```
module com.example.gui {
  requires javafx.controls;
}
```

This time, `javac` will execute successfully. 

However, notice that we only required `javafx.controls`, and not `javafx.graphics` as the error messages mentioned.

The reason is that if you use the `javafx.controls` module, most likely, you'll use the `javafx.graphics` too, so `javafx.controls` declares this module as a *transitive* dependency:
```
module javafx.controls {
  ...
  requires transitive javafx.graphics;
  ...
}
```

Of course, you can explicitly add `requires javafx.graphics` to your `module-info.java` file, but it's not required, you get it implicitly (automatically).

By the way, if you want to see all the modules of the JDK, the JDK installation directory contains a subdirectory named `jmods`. Conveniently, we can use the [jmod](https://docs.oracle.com/javase/9/tools/jmod.htm) tool to list the content of these JMOD files.

On my Windows machine, to see the details of the `javafx.controls` module, I execute:
```
jmod describe "C:\Program Files\Java\jdk-9\jmods\javafx.controls.jmod"
```

Moving on, our problems are not finished. If we run the program with:
```
java --module-path out --module com.example.gui/com.example.gui.QuoteFxApp
```

We'll get the following error:
```
java.lang.IllegalAccessException: class com.sun.javafx.application.LauncherImpl (in module javafx.graphics) cannot access class com.example.gui.QuoteFxApp (in module com.example.gui) because module com.example.gui does not export com.example.gui to module javafx.graphics
```

JavaFX needs access to our class `QuoteFxApp`, so we need to grant access explicitly by exporting the package `com.example.gui` this way:
```
module com.example.gui {
  ...
  
  exports com.example.gui;
}
```

However, this is just how JavaFX works, and probably we don't want any other modules to access our package. For cases like this, we can use a *qualified export*:
```
module com.example.gui {
  ...
  
  exports com.example.gui to javafx.graphics;
}
```

This way, only the listed modules (in this case only`javafx.graphics`) can access the package `com.example.gui`. Our application should work now:

![first run JavaFX](https://raw.githubusercontent.com/pluralsight/guides/master/images/7ea02f97-e7dd-4369-abe0-463bfc0c6422.png)

Having learned how to require/export other modules, integrating the programming quotes module and the GUI module should be easy.

First, in the module `com.example.programming`, export its package:
```java
module com.example.programming {
  exports com.example.programming;
}
```

Compile the classes and package them as a JAR file once again. We can copy this JAR to a `lib` directory for easy access.

Next, modify the class `QuoteFxApp` to use the `ProgrammingQuotes` class:
```java
...
import com.example.programming.ProgrammingQuotes;
...
public class QuoteFxApp extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception {
        ...

        Label label = new Label(new ProgrammingQuotes().getQuote());
        ...
    }

    ...
}

```

Require the `com.example.programming` module:
```
module com.example.gui {
  ...
  
  ...
}
```

Compile the `com.example.gui` module with the module path option so it can locate the `com.example.programming` module (which in this case, it's placed in the `lib` directory):
```
javac -d out --module-path ../lib module-info.java com/example/gui/QuoteFxApp.java
```

Finally, run the application using once again, the module path (remember, the path separator character for Windows is `;`. Use `:` for Linux/Mac):
```
java --module-path out;../lib --module com.example.gui/com.example.gui.QuoteFxApp
```

![second run JavaFX](https://raw.githubusercontent.com/pluralsight/guides/master/images/b4f57096-a43b-4cc8-934e-32d3ac4dc0c8.png)

Notice that if you omit the `lib` directory from the module path, the application won't run:
```
java --module-path out --module com.example.gui/com.example.gui.QuoteFxApp
Error occurred during initialization of boot layer
java.lang.module.FindException: Module com.example.programming not found, required by com.example.gui
```

This, as said before, solves in part the JAR/classpath hell problem.

# Using Services with the ServiceLoader
Let's say we now want to show math quotes in addition to programming quotes. We could create another module for math quotes and modify the GUI module to require it. 

We can also create an interface for both types of quotes. In the Java Platform Module System, this will give us the possibility to abstract the mechanism for matching up service interfaces with implementations using the [service locator pattern](https://en.wikipedia.org/wiki/Service_locator_pattern).

This works by using the [service-provider loading facility](https://docs.oracle.com/javase/tutorial/ext/basics/spi.html) that Java provides with the [ServiceLoader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html) class.

To use this mechanism, you need an interface, abstract class, or even a concrete class to act as the type of the service, an implementation or subclass, and use the class `ServiceLoader` to load all the implementations found.

Traditionally, this mechanism uses configuration files in the `META-INF/services` directory of the JAR to register implementations, but in the JPMS, this is done in the `module-info.java` file. Let's see it in action.

Create a module for the interface (generally, this module will contain all your API classes):
```java
package com.example.quote;

public interface Quote {
    String getQuote();
}
```

Export the interface's package in the `module-info.java` file:
```java
module com.example.quote {
  exports com.example.quote;
}
```

And Compile and package as usual:
```
# Compilation
javac -d out module-info.java com/example/quote/Quote.java

# Packaging (placing the jar in the common lib directory)
jar cvf ../lib/quote.jar -C out .
```

Now modify the `ProgrammingQuotes` class to implement the interface:
```java
...
import com.example.quote.Quote;

public class ProgrammingQuotes implements Quote {
    ...
}
```

Don't forget to add the interface module to the `module-info.java` file:
```java
module com.example.programming {
  requires com.example.quote;
  ...
}
```

And now, instead of exporting the package of the module, with the help of *provides with*, you can indicate that this module provides an implementation of the `Quote` interface:
```java
module com.example.programming {
  requires com.example.quote;
  
  provides com.example.quote.Quote
    with com.example.programming.ProgrammingQuotes;
}
```

Next, compile and package:
```
# Compilation
javac -d out --module-path ../lib module-info.java com/example/programming/ProgrammingQuotes.java

# Packaging (placing the jar in the common lib directory)
jar cvf ../lib/programming-quote.jar -C out .
```

But before proceeding, have you notice that we're not exporting the package of this module?

We're just declaring the interface and its implementation (both with their fully-qualified name).

The implementation is encapsulated in a non-exported package. This is generally the case because the point is to hide implementation details, the consumer only knows about the interface, not the implementation.

Talking about the consumer, in the GUI module, you use the `ServiceLoader` class like this:
```java
...

import com.example.quote.Quote;
import java.util.ServiceLoader;

public class QuoteFxApp extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception {
        primaryStage.setTitle("Quotes");
        
        Label label = new Label("NO QUOTE");

        ServiceLoader.load(Quote.class)
                .forEach(service -> label.setText(service.getQuote()));

        Scene scene = new Scene(label, 500, 200);
        ...
    }

    ...
}
```

The method `ServiceLoader.load(Class<T>)` returns a `ServiceLoader` instance that implements `Iterable`, so we can iterate over all the discovered implementations of the provided interface.

Next, in the `module-info.java` file, you just have to indicate that the module will require the module that contains the interface and it will *use* its implementations:
```java
module com.example.gui{
  ...
  requires com.example.quote;
  
  uses com.example.quote.Quote;
}
```

Compile and run like before:
```
# Compile
javac -d out --module-path ../lib module-info.java com/example/gui/QuoteFxApp.java

# Run (remember to change the path separator if needed)
java --module-path out;../lib --module com.example.gui/com.example.gui.QuoteFxApp
```

The app will work as before, but this time, it doesn't depend on the implementation, just the interface. If you want to add another implementation, just drop another module like `com.example.programming` into the `lib` directory and it will be discovered by the `ServiceLoader` class automatically.

# What about Maven?

Here's commonly asked question: Is there an overlap between Maven and JPMS? 

The answer is no; they complement each other.

While modularization is more about encapsulation and visibility (i.e. deciding which packages can be seen outside a module/JAR), Maven is about dependency management and compiling code into artifacts. The two things work at different levels.

On the topic of dependency management, every Maven artifact has three parts:
- A group ID, which uniquely identifies the project it belongs
- An artifact ID, which is its name
- A version

The JPMS does not know about versions, and depending on naming conventions, the name of a Java module can be the same as the artifact ID, the union of the group and artifact ID, or something completely different.

The bottom line is that because of this, we cannot directly associate a Java module with a Maven POM dependency/artifact. Basically, this means two things:
- Maven cannot generate the `module-info.java` file.
- In a modular application, adding a dependency involves two steps now:
  1. Adding the dependency to the POM file as always. This dependency can be modularized or not, it doesn't matter. Even if it is not modularized, it becomes a module automatically.
  2. The dependency's module is added to the `module-info.java` file of the project.
  
For example, let's assume our project consists of two modules, one for the API and one for the GUI, which depends on the API module.

By the way, we can structure our project with [Maven modules](http://books.sonatype.com/mvnex-book/reference/multimodule.html), at the high-level, the concept is basically the same. This is how the parent `pom.xml` would look like:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <groupId>org.example</groupId>
  <artifactId>my-parent</artifactId>
  <packaging>pom</packaging>
  <version>1.0</version>
  <name>Parent project</name>
  
  <modules>
    <module>api</module>
    <module>gui</module>
  </modules>
  
  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.7.0</version>
          <configuration>
            <source>9</source>
            <target>9</target>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  
  <!-- 
    ...
  -->

</project>
```

Nothing different from what we are used to, except maybe for the `maven-compiler-plugin` configuration. Make sure to use at least, version 3.6.1.

The directory structure of the project can be like the following:
```
|─ api
|   |─ pom.xml
|   |─ src
|      |─ main
|         |─ java
|            |─ module-info.java
|            |─ my
|               |─ example
|                  |─ api
|─ gui
|   |─ pom.xml
|   |─ src
|      |─ main
|         |─ java
|            |─ module-info.java
|            |─ my
|               |─ example
|                  |─ gui
|- pom.xml
```

For the API module, the `pom.xml` will look like this:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <parent>
    <groupId>org.example</groupId>
    <artifactId>my-parent</artifactId>
    <version>1.0</version>
  </parent>
  <artifactId>api</artifactId>
  <packaging>jar</packaging>
  <name>API project</name>
  
  <dependencies>
    <!-- 
      ...
    -->
  </dependencies>
  
  <!-- 
    ...
  -->

</project>
```

This would be the content of the `module-info.java` file:
```java
module my.company.api {
  exports my.company.api;
  // ...
}
```

Now, in the GUI module, we have to add the dependecy to the API project to its `pom.xml`:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <parent>
    <groupId>org.example</groupId>
    <artifactId>my-parent</artifactId>
    <version>1.0</version>
  </parent>
  <artifactId>gui</artifactId>
  <packaging>jar</packaging>
  <name>GUI project</name>
  
  <dependencies>
    <dependency>
      <groupId>org.example</groupId>
      <artifactId>api</artifactId>
      <version>1.0</version>
    </dependency>
    <!-- 
      ...
    -->
  </dependencies>
  
  <!-- 
    ...
  -->

</project>
```

As well as in `module-info.java`:
```java
module my.company.webapp {
  requires my.company.api;
  // ...
}
```

This way, when it's time to build the project (for example, with `mvn package`), the Maven compiler plug-in will set up the module path so that the Java compiler can work correctly.

# Conclusion
The module system has been introduced to Java to promote stronger encapsulation and better design, while making Java a more flexible and future-proof language.

There are features like [incubator modules](http://blog.takipi.com/how-java-9-incubator-modules-will-change-the-future-of-java/) that will bring new APIs to us while they progress towards either finalization or removal in a future release, giving us the change to provide input. This also fits pretty well with the [proposed fast release cycle for Java](https://mreinhold.org/blog/forward-faster).

In this tutorial, you have learned the basics of the JPMS, like the benefits of modules, how they work, and how they integrate with a tool like Maven.

Of course, there are several more things to learn, like [custom runtime images](https://blogs.oracle.com/jtc/creating-custom-jdk9-runtime-images), [how to migrate classpath projects to modules](https://dzone.com/articles/migrating-to-java-9), or how to [unit test a modular project](https://stackoverflow.com/questions/41366582/where-should-i-put-unit-tests-when-migrating-a-java-8-project-to-jigsaw).

The best book I have read about JPMS is [Java 9 Modularity](https://javamodularity.com/) by [Sander Mak](https://twitter.com/sander_mak) and [Paul Bakker](https://twitter.com/pbakker). I totally recommended it. Also, [The Java Module System](https://www.manning.com/books/the-java-module-system) by [Nicolai Parlog](https://twitter.com/nipafx) is a book that, at the time of writing this guide, is currently in progress yet looks very promising. Definitely look to these resources if you find yourself using Java 9 and JPMS in the future.

You can find the code of the sample JavaFX with ServiceLoader and Maven apps on this [GitHub repository](https://github.com/eh3rrera/getting-started-jpms).
___
I hope you found this guide educational and engaging. Please leave your comments and feedback in the discussion section below. Thanks for reading! 