This tutorial describes how to write and add a service to a licas server, before invoking it through the REST interface. 

<h2>Summary</h2>
<p>The software package <a href="http://licas.sourceforge.net" target="_blank">licas</a> (lightweight Internet-based communication for autonomic services) is a Java-based framework that allows a user to build distributed service-based networks that can also self-organise/self-optimise. Functionality is provided to allow for REST or XML-based RPC message passing and dynamic linking between services. Web services invocation and autonomic management are also provided for. The framework is very lightweight and the architecture and adaptive capabilities through dynamic linking, add something new that is not available in other similar systems. The licas system is mostly a p2p server and provides quite a lot of AI algorithms for more intelligent processing of information. It is also about providing a platform on which to run services that can provide some functionality for a user. Therefore, the primary architecture would be an SOA, or something similar.</p>

<h2>Starting a Server</h2>
<p>To use the licas framework, you start a server running, write and add a service to it and then invoke methods on the service. The framework provides a default server called HttpServer and its embedded Enterprise Service Bus that manages most of the interactions. The server is an HTTP server that you can start simply by running the main program, or ideally, extend it in your own code and start it from there. It can be password protected, or left open and configured using admin scripts. There are two base classes that are used to run and manage the server:<br/>
<br/>
•	The HttpServer class is the default server that loops continuously to accept incoming HTTP request calls and replies with HTTP responses. There is a second server class called ESB that is loaded onto the HttpServer. ESB is a type of enterprise service bus and you in fact interact with that to access any of the running services. The server itself only receives message requests and gives out low level information. The HttpServer can be statically accessed as well, which makes practical sense if you are running your program on the same JVM and do not want to bother making every call through the calling mechanism.<br/>
•	The ESB is the object that stores and interacts directly with the services. It provides all of the functionality, while the parent server class mostly accepts http requests and passes them on. For security reasons then, there is no static access and you need to know the password. Thus, any services that are stored also cannot be accessed directly, without knowing the server password as well. This provides a good level of protection at the base of the server.<br/>
•	The Run_HttpServer class configures and starts the server running on a JVM. You might typically add a copy of this class to your own code and set it as Main to start a server. Note that both the http and the https protocol can be used.<br/>
<br/>
The easiest way to start a server is to run the batch file provided with the <a href="http://licas.sourceforge.net" target="_blank">download</a>. Alternatively, the free All-in-One <a href="http://distributedcomputingsystems.co.uk/licas.html" target="_blank">GUI</a> includes a built-in server and access to all of the main functionality. The Run_HttpServer class has a command line interface that you can configure to change the run settings. The batch file and the user guides give more details on this.</p>

<h2>Writing a Service</h2>
<p>The licas system is about providing a platform on which to run services for a user. The base service classes provided by the package are the Service, Behaviour or the Auto classes. These are all abstract and so you would extend one and implement the abstract methods to write your own service. There are a number of examples in the main package – DataService and InformationService, for example. To write your own service, you can extend any of these abstract classes, or any of the fully implemented versions. For a very basic service, for example, you could write the following:</p>
<br/><br/>
<pre><code class="lang-javascript hljs">
<span class="hljs-string">public class MyInfoService extends Service</span>
<span class="hljs-string">{</span>
<span class="hljs-string">/** The logger */</span>
<span class="hljs-string">private static Logger logger;</span>
        <br/>
<span class="hljs-string">static</span>
<span class="hljs-string">{</span>
<span class="hljs-string">// get the logger</span>
<span class="hljs-string">logger = LoggerHandler.getLogger(MyInfoService.class.getName());</span>
<span class="hljs-string">logger.setDebug(false);</span>
<span class="hljs-string">}</span>
<br/>
<span class="hljs-string">public TestService() throws Exception</span>
<span class="hljs-string">{</span>
<span class="hljs-string">super();</span>
<span class="hljs-string">}</span>

<span class="hljs-string">public TestService(String thisPassword, String thisAdminKey) throws Exception</span>
<span class="hljs-string">{</span>
<span class="hljs-string">super(thisPassword, thisAdminKey);</span>
<span class="hljs-string">}</span>

<span class="hljs-string">public TestService(String thisPassword, String thisAdminKey, Element adminXml)  throws Exception</span>
<span class="hljs-string">{</span>
<span class="hljs-string">super(thisPassword, thisAdminKey, adminXml);</span>
<span class="hljs-string">}</span>

<span class="hljs-string">public String GET() throws Exception</span>
<span class="hljs-string">{</span>
<span class="hljs-string">String dataValue;                           //the data value</span>

<span class="hljs-string">dataValue = "";</span>
<span class="hljs-string">dataObj = getNextValue(…);</span>
<span class="hljs-string">if (dataObj != null)</span>
<span class="hljs-string">{</span>
<span class="hljs-string">if (dataObj instanceof byte[])</span>
<span class="hljs-string">{</span>
<span class="hljs-string">dataValue = (TypeConst.BINARYFILE + Const.RNDSEP + new String((byte[])dataObj));</span>
<span class="hljs-string">}</span>
<span class="hljs-string">else if (dataObj instanceof Element)</span>
<span class="hljs-string">{</span>
<span class="hljs-string">dataValue = (TypeConst.XMLFILE + Const.RNDSEP +
			XmlHandler.xmlToString((Element)dataObj));</span>
<span class="hljs-string">}</span>
<span class="hljs-string">else</span>
<span class="hljs-string">{</span>
<span class="hljs-string">dataValue = (dataType + Const.RNDSEP + String.valueOf(dataObj));</span>
<span class="hljs-string">}</span>
<span class="hljs-string">}</span>

<span class="hljs-string">return dataValue;</span>
<span class="hljs-string">}</span>

<span class="hljs-string">...</span>
<span class="hljs-string">}</span>
</code></pre>
<br/><br/>
<p>The logger is the default licas logger. The constructors can all pass the parameters to the parent service class. This service has then implemented the GET method, which is of interest for the REST interface. If you type the service address into a browser, then the request is directed to the get method of the related service, not a web page. The service can then perform any functionality in its GET method and return any String result. The service will automatically parse this first to check for a data type, where Const.RNDSEP is used as the default tokenizer (data type – reply). The browser can then automatically display the reply, for example and so this is a quick way to ask a service to perform some function. 
<br/><br/>
The server can also be asked directly to return a file from its web directory. Therefore:
<code class="lang-javascript hljs">
http://192.168.2.2:8080/html/default.html
</code>
would ask the server to return the default.html file from its web html directory, but
<code class="lang-javascript hljs">
http://192.168.2.2:8080/service1
</code>
would ask service1 to return the result of its GET method.
</p>

<h3>Communication Protocols</h3>
<p>Licas uses an XML-RPC mechanism internally, but this tutorial describes the REST interface specifically. XML-RPC is maybe better for programmable interfaces or computer-to-computer interactions, allowing complex objects to be passed easily. The REST interface is supported with Resources and an Information Service, so that basic media types can be stored and returned upon request. It is also useful for the IoT possibilities. 
<br/><br/>
On the client side, when making a remote call, you would create the appropriate MethodInfo object and execute it on a CallObject. The MethodFactory class is useful here and provides 3 static methods that will create the info object for you – createMethodCall, createRestCall or createSoapCall. There are lots of other parameters that can be set, but those are the main components. The address of the service to invoke can be a URI description, or for local interactions, it can be a direct reference to the object itself. The system will check for this and use method invocation when appropriate. 
</p>

<h3>Create a Service Path Handle</h3>
<p>This is typically used by the internal XML-RPC process, but it is also the URI path definition for any service that runs on a server. It has a specific format that is created by the Handle class. The address of a service uniquely defines it on a server and over the Internet and includes a number of parts. Each of these parts is stored inside of an XML element of a specific type. The outermost element is simply called 'Handle'. Then it is required to define the server address and the service name. The first part is the URL of the server that is hosting the service. This is stored in a 'U' element. Each base service running on a server is then required to have a unique UUID. This is stored in an 'S' element. Nested services can also be declared by adding ‘S’ elements, one after the other. If the path is for a method invocation, then a final element that stores the method name is called 'M', although this part is typically added by the system. A full path description without a method could look something like:
<br/><br/>
<code class="lang-javascript hljs">
<Handle><U>http://123.4.5.6:8888</U><S>Service1</S></Handle>
</code>
</p>

<h2>The REST Interface</h2>
<p>While XML-RPC is the internal method, the server can also receive and parse a REST interface, with key-value pairs. The user guide describes how the default MethodInfo object can be described in REST. For a client to send a REST request, instead of calling MethodFactory.createMethodCall for the XML-RPC interface, you would use MethodFactory.createRestCall to get the REST client object. The code section might typically include the following:
<br/><br/>
<pre><code class="lang-javascript hljs">
CallObject call = new CallObject();
MethodInfo methodInfo = MethodFactory.createRestCall((String)endpoint, (String)rest_request);
String reply = (String)call.call(methodInfo);
</code></pre>
<br/><br/>
The request is invoked on the endpoint address and the reply is a String that can be processed as required. Note that this works the same way for invoking a licas server or any other external source, such as a web service.
</p>

<h2>Conclusions</h2>
<p>The licas framework offers a rich set of features and a flexible communication mechanism, to allow you to invoke local or remote objects of different types. It is relevant to the IoT, with an Information Service and related resource objects and the related communication mechanisms. Its AI packages then add something new that would allow for research and development in this new technological area. Licas also works on Android now.</p>
<br/>
<br/>




