
## Introduction and Background
I have been writing many guides and tutorials about IoT programming on the device itself, or the programming of web services separately, however since a while I had been finding the best possible way to allow the IoT devices to be able to communicate, while also keeping things really very easy and simple. This guide is for you, if you:

1. Are an IoT developer or work with IoT folks for providing communication support for the connected devices. 
2. Are building web services for devices.
3. Are an ASP.NET developer.
4. Anyone else... Someone like, who wants to build a native HTTP web service. 

I will provide a few of the tips as to why you should consider _using ASP.NET Web API as the main framework for programming the web services for the IoT_ connected devices. One of the facts that I believe in now is that, people have started to take the **wrong** meaning of the term, IoT. They believe that IoT is something new and special, in this post I am going to show you why they are wrong. 

## Web Services for IoT
Since a few years back, everything has required a network connection to work in the server-client architecture. In this guide my focus would be to deliver the data on the IoT devices and a bunch of other tips that you would be interested in. But first let us dig deeper and understand what made the network connected device communication very important in modern day. The factors that make network requirement a must-have are the factors that can improve security of your application, performance of your applications, decrease the memory requirements on the devices and much more. 

Besides the usages, it is also better to consider using a better framework for building the web services. I have been told many times that IoT requires a lot of considerations, IoT is tough etc. etc. But the thing is, it is the same framework as embedded computing was. In the old days, it was not given a particular name and programmers called it embedded computer, however recent programmers do not call it the way it was, instead the term IoT (Internet of Things) device is an appropriate term for these devices. In my own experience I have found that building an ASP.NET Web API is really very helpful and straight forward. I have found building ASP.NET Web APIs very much simple and helpful in many cases as compared to building the WCF applications and web services for the connectivity of the devices. 

### Why use ASP.NET?
In this section, I will talk about a few benefits of using ASP.NET Web API instead of other web services frameworks. Historically, ASP.NET has been used for building and deploying web applications in HTML, CSS and JavaScript. However, time changes and ASP.NET started to roll out a new framework for programming called, ASP.NET Web API. It follows the same architectural design as ASP.NET MVC and provides a very compact benchmark for HTTP communication, rather than having to submit and share HTML documents. All much that I have learnt from my experience is that using ASP.NET Web API for the web services framework can really ease many pains, such as, having to create the clients and authorizing the clients that can communicate. This problem came while programming WCF web services a lot. One of the problems that I came to see in WCF was that it required to generate certificates for the clients and then using those service references, you will be able to consume the services. However, what if you required to consume a service from some microcontroller? Or if you wanted to consume the service from Android device, where .NET framework is not available. In such cases, WCF framework doesn't work and you would have to find a way to work around this problem such as using ASP.NET web applications... Which leads us back to where we started before. Besides instead of just a chatter why don't I let you review what [MSDN](https://msdn.microsoft.com/en-us/library/jj823172(v=vs.110%29.aspx) has to say in this matter, 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3ef0e160-9567-48b1-8f25-19d9057b2263.gif)

Also note that WCF is a complex programming framework and requires a bunch of considerations and learning before you can start writing web services (and note that it is a web services framework for .NET applications only), where as ASP.NET Web API doesn't even require you to learn HTML, CSS and JavaScript. You just need to the HTTP protocol in order to make things work. That is not just it, there are a few other benefits that require a separate talk topic, so let's give them one. 

#### Performance
Performance factor of both WCF and ASP.NET is good and they don't really cause any problem in this case. On the other hand, it can be a bad programming that would cause a performance factor to fall in both of them. However, what I want to talk here is that the codes in API calls can be as simple as, 

``` csharp
 public string Get() {
    // Return the list of the data
    return dataList;
 }
```
Performance depends on a bunch of factors only:

1. CPU and RAM resources available for the programs to run; _ASP.NET Web API can be hosted in a program_.
2. Network bandwidth and throughput. 
3. Other services running in the background. 
4. Time required by data engines to return the data, or to load the data from the memory or files. 

Rest of the stuff is performed by the ASP.NET Web API to serialize the data to JSON format and send the data over the network. Most of the times, serialization and deserialization of data can also take time, luckily, ASP.NET Web API uses [Newtonsoft.Json](http://www.newtonsoft.com/json) API to perform data (de)serailization actions and which are notably fast as compared to the rest of the stuff. 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/571356bf-5224-444a-a900-c49709100ade.png)

As clearly seen, even the entry level and outgoing level doors of ASP.NET Web API are fine tuned to not give any performance factor a hard time. This is what makes ASP.NET Web API a very powerful framework to create web services on the top of HTTP protocol. Unlike WCF, where you had to upload and run entire project, no matter how small the memory benchmark but a very huge amount of service was requried to run and then it would handle the request on TCP/IP protocol, in this case, HTTP is based on TCP and doesn't cause a problem in the communication. 

#### Self-hosting of services
If you think that just the way WCF required to run entire project, ASP.NET Web API would require the same. **You're wrong!** ASP.NET Web API is an open source project and you can host the project in a minimal **console application** too. Which allows the project to just consume the RAM and CPU resources for the required request handling and not for other services that are not at all required. 

Yes, that is right, entire code to host your Web API in the console program is the following, 

```
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
config.Routes.MapHttpRoute(
    "API Default", "api/{controller}/{id}", 
    new { id = RouteParameter.Optional });

// Start the service. 
using (var apiServer = new HttpSelfHostServer(configuration))
{
    apiServer.OpenAsync().Wait();
    
    // Wait for the server manager to close it. Until then, anyone can connect.
    Console.WriteLine("Press Enter to quit.");
    Console.ReadLine();
}
```

This is just the code required to host the service, you can host it anywhere. Rest of the job is your ASP.NET Web API's code, where you actually handle the requests. This code would forward the request to the API Controllers and they will perform the actions based on the HTTP verb being used. This enables your web services to run in a very low memory, and consume very less amount of CPU clock ticks. These are a few of the things that make ASP.NET Web API my personal favorite framework to build web services for the connected devices. 

#### Cross-platform support
WCF framework had one problem: It was hard to consume their services on platforms like, Android, iOS, microcontrollers etc. That problem can be removed by using native protocols such as TCP/IP or HTTP themselves. The benefit of native protocols would be that any device will be able to implement these protocols at their ease and there will be no issue at all. Unlike WCF, native protocols won't require any client-being ceritificate to be issued from the server. ASP.NET Web API can be hosted in a small environment, it can also be published the servers where your enterprise libraries and cloud solutions can provide resilient and fast responses to the clients. This way, you can allow communication with any client over the HTTP protocol. 

Since TCP/IP and HTTP protocols can be programmed on every platform now a days, such as:

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

Which means that a native service would be  consumed on every platform and you don't have to worry about the underlying frameworks at all. Besides, it also provides you with access to native GUI and native UX guidelines because you can access the data in the application itself and render it on the screen, speak it out or print the data. **Everything is possible, when you go cross-platform**. 

### Building a small web service
ASP.NET Web API controllers are used to target and control how the URL mapping is performed to provide the results and responses to the clients. To create a new Controller, `System.Web.Http.ApiController` class is used as the base class for the object. Later, the HTTP verb functions are set for that route. Routing is set using the `Route` attribute of ASP.NET Web API framework. Rest of the stuff is just based C# programming, to return the values from the functions and accept values to the functions as parameters. 

```
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

This is a complete Web API for your controllers. Although, this doesn't support much of the features of the API, but you get the idea of mapping of the Web API and how to create on for your application project. Notice how simple this program is, the rest of the job is done by the handling method where you define how your routines are defined for the URL mapping. 

#### Writing other HTTP URL helpers
Since the program is very simple, I wanted to show you how you can add more URLs to be handled in this specific controller of your application. Note that we have routed our application to the HTTP requests using the `Route[("path")]` attribute over the actions of the controllers. We can use them on other actions, to define different routes in the same controller. Have a look above, and see that are having an HTTP GET action handler. What if we wanted to handle another GET request? For that, we can change the URL where we want to map that, and still handle that request under this controller. Such as like this, 

```
[Route("api/microcontrollers/anotherurl")]
public string Get() {
   return "Hello world, from another URL.";
}
```

Now if we will make the request to this URL (_localhost:12345/api/microcontrollers/anotherurl_) that will be mapped to action and the result will be generated from this action. This makes it really helpful and easy to handle multiple URLs and actions under the same controller, without having to create a separate module for each of the URL that is going to be mapped or _raise an error_. 

### Connecting to the web service
As we have known, our service is basic HTTP based service and which can be consumed easily using any HTTP client. Most of the frameworks provide support for HTTP communication, every operating system has APIs that allow HTTP based communication over the network. So, basically, you won't have to worry about anything at all before working on this. HTTP clients are provided in C#, Java, C++, Python and almost every other programming language for the need of the network communication in the applications. In this guide, I am going to walk you though .NET framework's HTTP clients that allow HTTP communication with your services. In .NET framework, `HttpClient` object is majorly used (in contrast to others such as `WebClient` etc) and this provides your applications access to the network communication and allows you to send requests with the HTTP verbs such as `GET`, `POST` etc. You can also publish the content to the server after serializing the data for the publishing processes. 

Like other `IDisposable` objects, `HttpClient` also allows you to leave the rest of the memory cleanup and disposing of the resources to the .NET framework. This way, you will be able to focus on the network programming and not on the memory cleanup most of the times. 

Let me now show you how to connect to the services using `HttpClient` object in .NET applications. Note that you can use `HttpClient` in any project, any application if you can target `System.Net.Http` namespace in your application. So the project's main function of the program would look something like this:

```
// // Namespace import
using System.Net.Http;

// Deep inside the function
using (var client = new HttpClient()) {
   // Use the API here.
}
```

This is the template that you will be using for consuming the application service. So the very basic program to actually get the Hello world message from the API would be like this, 

```
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
This way, you get to be able to access the content. The rest of the stuff is also similar, and `HttpClient` allows you to use other functions to perform other HTTP requests such as `POST`, where you get to publish some content to the servers. Note that publishing requires a bit of complex methods where you also have to wrap and pass the data in the form of bytes array or other similar streams to the server which are then loaded and read on the server side. In case of microcontrollers, that doesn't make much sense because you are not going to stream the media (unless you really want to!) and in that case, you can surely check out one of articles about network programming in .NET framework. 