## Guide to make your Java applications extensible using Plugin Framework for Java (PF4J)

This document aims to provide a beginners level tutorial based guide for using PF4J in with your Java application. 

> [PF4J](https://github.com/decebals/pf4j) is an open source (Apache license) lightweight plugin framework developed by [Decebal Suiu](https://github.com/decebals). It is quite simple but powerful. With PF4J you can easily transform a monolithic java application to a modular java appication extensible through plugins.


### Overview

A plugin is a way for a third party to extend the functionality of an application.

#### Components

- **Plugin** is the base class for all plugins types. Each plugin is loaded into a separate class loader to avoid conflicts.
- **PluginManager**  is used for all aspects of plugins management (loading, starting, stopping).
- **ExtensionPoint** is a point in the application where custom code can be invoked. It's a java interface marker. Any java interface or abstract class can be marked as an extension point (implements ExtensionPoint interface).
- **Extension**  is an implementation of an extension point. It's a java annotation on a class.





### References
- https://github.com/decebals/pf4j/blob/master/README.md
- https://groups.google.com/forum/#!forum/pf4j
