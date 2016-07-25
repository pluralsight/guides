## Introduction and Background
I have been writing many guides and tutorials about Internet of Things (IoT) programming on smart devices themselves or the programming of web services separately. However since a while I had been finding the best possible way to allow the IoT devices to be able to communicate, while also keeping things really very easy and simple. This guide is for you, if you are: 

1. An IoT developer or work with IoT folks for providing communication support for the connected devices. 
2. Interested in building web services for devices.
3. An ASP.NET developer.
4. Anyone else, such as someone who wants to build a native HTTP web service. 

My goal in this tutorial is to convince you to consider using ASP.NET Web API as the main framework for programming the web services for the IoT-connected devices. 

In my opinion, the **wrong** definition of IoT has started becoming more popular; people believe that IoT is something new and special, but this notion is incorrect. IoT devices have been the same embedded devices, mobile devices and other desktop applications that we've seen for many years. Yet, currently they are more powerful. They provide more features but there is no _new_ science behind them. 

## Web Services for IoT
A few years ago, we witnessed a significant shift in technology: everything began requiring a network connection to work in the server-client architecture. 

In this guide, my focus is on delivering user data on the IoT devices and teaching numerous other pertinent factors of web-based IoT programming. But first let's dig deeper and understand what made the network connected device communication so important to everyday life. 

The factors that make network requirement a must-have are the factors that can improve security of your application, performance of your applications, decrease the memory requirements on the devices and much more. 

Besides the usages, it is also better to consider using a better framework for building the web services. I have been told many times that IoT requires a lot of considerations, that IoT is tough, and so on. But the thing is, **it is the same framework as embedded computing was**. In the old days, the framework was not given a particular name, so programmers called it these systems Embedded Systems. Recently, programmers have stopped calling them Embedded Systems, opting for the IoT label instead. 

In my own experience, I have found that building an ASP.NET Web API is very helpful and straightforward when compared to building the WCF (Windows Communications Foundation) applications and web services for the connectivity of the devices. 

Web services (or the server-client framework) should be considered because they require marginal programming. Furthermore, in case of any trouble, your clients don't have to come to you and get the devices fixed. Web services allow customers to take advantage of quick access using the worldwide web.

Some other advantages of embedded web services:

1. All of the source code remains on your hand.
2. You can fix, monitor, update and publish new features as they come live from your development teams. 
3. Clients' devices don't stay broken if there is a problem. 
4. Devices with few hardware resources can also work because most of the processing is done on the server. 
5. Anyone can connect to these services.

There are countless frameworks used for programming web services. Embedded programming requires you to write software for each device and then publish it to that device by deploying it. Web services (or server-client framework) don't require customers to bring their problems to the manufacturer; clients consume the resources from the server. Of the problems that occur, most happen on the server and not on the client side. Thus, once a problem is faced, the team has to fix it **in one place** and not on every device. 

### Why use ASP.NET?
In this section, I will talk about a few benefits of using ASP.NET Web API instead of other web services frameworks. 

Historically, ASP.NET has been used for building and deploying web applications in HTML, CSS and JavaScript. However, ASP.NET started to roll out a new framework for programming called ASP.NET Web API. It follows the same architectural design as ASP.NET MVC and provides a very compact benchmark for HTTP communication, rather than having to submit and share HTML documents. 

Based on experience, I believe that using ASP.NET Web API for the web services framework can ease many pains, such as having to create the clients and authorizing the clients that can communicate. This problem arose frequently while programming WCF web services. 

Another common problem with WCF programming was that it required coders to generate certificates for the clients and then, using those service references, clients would be able to consume the services. However, what if you required to consume a service from some microcontroller? Or if you wanted to consume the service from Android device, where .NET framework is not available? In such cases, the WCF framework is not feasible, so you'd need a workaround.

Also note that WCF is a complex programming framework and requires a bunch of considerations and learning before you can start writing web services (and note that it is a web services framework for .NET applications only), whereas the ASP.NET Web API doesn't even require you to learn HTML, CSS, or JavaScript. You just need to understand the HTTP protocol in order to make things work.

This leads us back to ASP.NET. Just look at what [MSDN](https://msdn.microsoft.com/en-us/library/jj823172(v=vs.110%29.aspx) has to say in this matter, 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3ef0e160-9567-48b1-8f25-19d9057b2263.gif)

Next, we'll consider some more detailed comparisons of WCF and ASP.NET.

#### Performance
The performance of WCF and ASP.NET is good. Neither framework causes any problems in this case. Let's consider a simple API call. 

``` csharp
 public string Get() {
    // Return the list of the data
    return dataList;
 }
```
Performance depends on certain factors:

1. CPU and RAM resources available for the programs to run; _ASP.NET Web API can be hosted in a program_.
2. Network bandwidth and throughput. 
3. Other services running in the background. 
4. Time required by data engines to return the data, or to load the data from the memory or files.  

The rest of the stuff is performed by the ASP.NET Web API to serialize the data to JSON format and send the data over the network. Most of the times, serialization and deserialization of data can also take time; Luckily, ASP.NET Web API uses the [Newtonsoft.Json](http://www.newtonsoft.com/json) API to perform data (de)serailization actions and which are notably fast. 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/571356bf-5224-444a-a900-c49709100ade.png)

Even the entry level and outgoing level doors of ASP.NET Web API are fine tuned to not give any performance factor a hard time. This is what makes ASP.NET Web API a very powerful framework to create web services on the top of HTTP protocol. 

Unlike WCF, which used an often-inefficient amount of service to upload and run entire projects, no matter how small the memory benchmark, and handled it using TCP/IP, HTTP is based on TCP and doesn't cause problems in communication. 

#### Self-hosting of services
If you think that ASP.NET and WCF both require running entire projects in the same way, **you're wrong!** 

ASP.NET Web API is an open source project and you can host the project in a minimal **console application** too. This hosting method allows the project to just consume the RAM and CPU resources for the required request handling and not for other services that are not at all required. 

Yes, that is right, the only code required to host your Web API in the console program is the following, 

``` csharp
// Namespace references
using System.Web.Http;
using System.Web.Http.SelfHost;            

/*
 * This goes in the main function
 * The main function is required, rest is optional.
 */
 
 // Create a new object with the address where you want to host the application at.
var configuration = new HttpSelfHostConfiguration("http://localhost:12345");

// Add the routing scheme, this is required. 
config.Routes.MapHttpRoute("Default", "api/{controller}/{id}", new { id = RouteParameter.Optional });

// Start the service. 
using (var apiServer = new HttpSelfHostServer(configuration))
{
    apiServer.OpenAsync().Wait();
    
    // Wait for the server manager to close it. Until then, anyone can connect.
    Console.WriteLine("Press Enter to quit.");
    Console.ReadLine();
}
```

This is the code required to host the service, which you can host anywhere. The rest of the job is working with your ASP.NET Web API's code, which you use to actually handle requests. This code would forward the request to the API Controllers and they will perform the actions based on the HTTP verb being used. This enables your web services to run on very low memory and consume few CPU clock ticks. 

These are a few of the things that make ASP.NET Web API my personal favorite framework for building web services for the connected devices. 

#### Cross-platform support
The popular WCF framework had another key problem; it was hard to consume services on platforms like Android, iOS, and microcontrollers. That problem was removed by using native protocols such as TCP/IP or HTTP themselves.

Native protocols allow all devices to implement these protocols at their ease without issues. Unlike WCF, native protocols don't require any client-being ceritificate to be issued from the server. The ASP.NET Web API can be hosted in a small environment, and it can also be published the servers where your enterprise libraries and cloud solutions can provide resilient and fast responses to the clients. This way, you can allow communication with any client over the HTTP protocol. 

TCP/IP and HTTP protocols can be programmed on every platform nowadays, such as:

1. Windows
 - Windows Runtime (Windows 8 and Windows 10 applications)
 - .NET framework
 - Native Windows services
2. Linux
 - Android
 - Ubuntu
 - Rest of the Linux distros.
3. Apple products
4. Microcontrollers

As such, a native service would be consumed on every platform, so you don't have to worry about the underlying frameworks at all. Besides, using a native service also provides you with access to the native GUI and the native UX guidelines because __you can access the data in the application itself and render it on the screen, speak it out or print the data__. Everything is possible, when you go cross-platform. 

### Building a small web service

ASP.NET Web API controllers are used to target and control how the URL mapping is performed to provide the results and responses to the clients. 

To create a new Controller, we use the`System.Web.Http.ApiController` class as the base class for the object. Later, the HTTP verb functions are set for that route. Routing is set using the `Route` attribute of ASP.NET Web API framework. Rest of the stuff is just based C# programming, to return the values from the functions and accept values to the functions as parameters. 

``` csharp
// Add the namespace imports
namespace Sample {

   // Add the route attribute, this would be accessible at: localhost:12345/api/microcontrollers/
   [Route("api/microcontrollers")]
   public class MicroControllers : ApiController { // Note the inheritance
      // Adding the HTTP helpers
      public string Get() {
         return "Hello, world!";
      }
      
      [Route("api/microcontrollers/{name}")]
      public void Post(string name) {
         dataModel._name = name; // Save the naem
      }
      
      [Route("api/microcontrollers/{id}")]
      public void Delete(int id) { 
         dataModel.Delete(x => x._id == id); // Sample function to delete the data.
      }
      
      /*
       * You can add other functions named with the HTTP verb
       * such as:
       * 1. PUT
       * 2. OPTIONS
       * etc.
       */ 
   }
}
```

This is a complete Web API for your controllers. Although this doesn't support many of the features of the API, hopefully you understand the advantages of mapping the Web API. Notice how simple this program is; the rest of the job is done by the handling method where you define how your routines are defined for the URL mapping. 

#### Writing other HTTP URL helpers
Since the program is very simple, I wanted to show you how you can add more URLs to be handled in this specific controller of your application. Note that we have routed our application to the HTTP requests using the `Route[("path")]` attribute over the actions of the controllers. We can use them on other actions, to define different routes in the same controller. 

Take a look above, and see the HTTP GET action handler. What if we wanted to handle another GET request? 

For that, we can change the URL where we want to map that, and still handle that request under this controller. Such as like this, 

``` csharp
[Route("api/microcontrollers/anotherurl")]
public string Get() {
   return "Hello world, from another URL.";
}
```

Now if we change the request to this URL (_localhost:12345/api/microcontrollers/anotherurl_), we will be mapped to an action and the result will be generated from this action. This makes it really helpful and easy to handle multiple URLs and actions under the same controller, without having to create a separate module for each of the URLs that is going to be mapped or _raise an error_. 

### Connecting to the web service
As we have known, our service is a basic HTTP-based service that can be consumed easily using any HTTP client. Most frameworks provide support for HTTP communication, and every operating system has APIs that allow HTTP-based communication over the network. 

So, basically, you won't have to worry about anything at all before working on this. HTTP clients are provided in C#, Java, C++, Python and almost every other programming language in order to facilitate in-application network communication. In this guide, I am going to walk you though the .NET framework's HTTP clients that allow HTTP communication with your services. 

The .NET framework usually uses the `HttpClient` object (contrast this with `WebClient`). This provides your applications access to the network communication and allows you to send requests with the HTTP verbs such as `GET`, `POST`, and so on. You can also publish the content to the server after serializing the data for the publishing processes. 

Like other `IDisposable` objects, `HttpClient` also allows you to leave the rest of the memory cleanup and dispose of the resources to the .NET framework. This way, you will be able to focus on the network programming and not on the memory cleanup most of the time. 

Let me now show you how to connect to the services using `HttpClient` object in .NET applications. Note that you can use `HttpClient` in any project, any application if you can target the `System.Net.Http` namespace in your application. 

So the main function of the program would look something like this:

``` csharp
// // Namespace import
using System.Net.Http;

// Deep inside the function
using (var client = new HttpClient()) {
   // Use the API here.
}
```

This is the template that you should use for consuming the application service. For instance, the basic program  for getting a Hello world message from the API would look like this, 

``` csharp
using (var client = new HttpClient())
{
    // It is recommended to add a base address, instead of pushing this to the HTTP request functions.
    client.BaseAddress = new Uri("http://localhost:12345/");

    // Get the response from the API
    var response = await client.GetAsync("api/microcontrollers/");
    
    // Check if the request succeeded.
    if (response.IsSuccessStatusCode)
    {
        // Get the message.
        var msg = await response.Content.ReadAsStringAsync();
        Console.WriteLine(msg); // Hello, world!
    }
}
```
This way, you can access the API remotely. The other ASP.NET basics are similar. For example, `HttpClient` allows you to use other functions to perform other HTTP requests such as `POST`, where you get to publish some content to the servers. Note that publishing requires a bit of complex methods where you also have to wrap and pass the data in the form of a bytes array to the server. This data is then loaded and read on the server side. 

In case of microcontrollers, that doesn't make much sense because you are unlikely to stream media (and thus use a bytes array). As such, you can surely check out one of the many articles about network programming in the .NET framework on microcontrollers. 

Thank you for reading. I hope you realize the many advantages of ASP.NET when compared to the traditional WCF framework. Furthermore, I hope that the similarities among the platforms indicate to you that the notion of IoT is not actually as "new" as it's made out to be!

Please add your comments and feedback in the discussion section below! Thanks again!