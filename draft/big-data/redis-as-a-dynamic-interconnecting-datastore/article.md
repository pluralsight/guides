
Introduction
So … What is Redis?
Using Redis As a Synchronized Cache Mechanism
Distributed Locking Mechanism at Your Disposal
Sample Context Interaction Implementation
Interprocess Communication With Redis
Sample Publisher / Subscriber Mechanism Implementation
In Conclusion




Introduction
	“Context and memory play powerful roles in all the truly great meals in one's life.”
	Anthony Bourdain, Chef

Yes, I know. He (Anthony) is talking about meals and we are talking about software development. But not all is different. Sometimes our data, passing through our software parts, goes through specific transformations, being passed around according to a very specific algorithm. The output of that data will eventually become our output, or “meal”, if you want to.




The output of each computation step will probably be one of the following:
The final output of our processing.
The input for the next processing step.

The current processing step we are in is usually referred to as our state. Any data we gathered up to this point in the computation is our state data.



Now, the state data might go through processing, at the current processing state, with different parameters each time (i.e. current date and time, user id). These parameters are referred to as context. It is very common that a message either contains it’s context or has a hint about how to resolve the context. If the context is being passed inside the message, then there is no real “resolving” step.


Each processing state might require access to the context of the data being passed to it, in order to fulfill its designed purpose.



So context resolving is the process through which a running software component receives the relevant context to the current computation it is requested to perform.

In some common cases, such as web applications, that context could be the session established per user, when he logged into our system or contain any metadata about the data being passed to us.

Working with software systems, these days, is not like several years ago. Various software components might change our context as time passes. These components need access to a shared resource which they can refer to, simultaneously, and consume (read / write) the context they so need.

Redis is a provider of such capabilities.
So … What is Redis?
	Redis, originally created by Salvatore Sanfilippo, was named “Redis” to reflect its original purpose to provide “Remote Dictionary Server”. It was initially released at GitHub at 2010 and had undergone many following releases since.


I will be providing, here, a birds eye view of Redis, as it is today (e.g. IMHO - More than a remote dictionary server).




You, the Redis consumer, will be hosting a Redis client in your application.
That Client, will communicate any request given to it, to the Redis server (or servers, when deployed as a cluster). This communication will be performed over a simple and efficient protocol, over TCP.

The requests we pass on to Redis will be mostly CRUD operations in the form <Key, Object>, where Object can take on the form of any supported data type, such as a string, a list, an ordered list or a set.

For example, the following information was persisted in Redis for demonstration (I am using the Open source Redis commander user interface hosted on docker):


A simple Key, Value pair.

A list of values associated with the same key. 

We can optionally define an expiration time for any key we store in Redis. This expiration time will be accurate to the millisecond (version >= 2.6).
Expired keys (and their values) will be deleted automatically once they’ve expired.

It’s important to note that what makes Redis very interesting as a centralized Cache / Datastore / Synchronized data provider is that it is blazing fast, mostly due to the fact that all data is persisted in-memory. By default, Redis will also persist all data to non-volatile storage (this setting can be configured), and replication and high-availability are also supported - though these features are not at the core of this article.

Let’s observe some use cases and see some code to better understand the technology.



Using Redis As a Synchronized Cache Mechanism
	A most common use-case for Redis is as a distributed cache.
Having more than one process in our application, Redis can be a “central point of reference” to all processes (optionally across more than one machine).

Having a central, single representation of our systems data allows us to safely implement our logic - “knowing” that at a given point in time, per input message - any process “looking” at a specific piece of information (e.g. context) - will “see” the same information.



At the diagram above, we can see two processes, to which input message #1 arrived, at the exact same time. Both processes are requesting the message context from Redis, and will receive the exact same context!

There is also a different situation. It might be the case that the output of a single process is the input for another. In such a case, the times t1 and t2 will be different (t2 > t1), resulting in an optionally different context for each process.




For example, assuming our computation began as a user logged into our system, before he performed any additional operation - we already “know” something about that user (i.e. its name, its login location and historical activities in our system).

Some of that information might be used across our computation, probably by different processes (hosted on different machines).

In such a case, we might do wisely to cache that context information in-memory and have it provided quickly to any consuming process.



That is very helpful indeed. As long as we can make sure that our data is properly synchronized.

Remember the statement I’ve made about having a single representation of our context at a given point in time? Well, it appears that making sure this is true, is not a simple a task when dealing with concurrent systems. Multiple processes might access the same data at the same time. Some might try to modify it.



Lets see how Redis allows us to deal with concurrent data consumption.
 


Distributed Locking Mechanism at Your Disposal
Redis supports and promotes the uses of the Redlock algorithm, in order to implement distributed locking of resources. The algorithm is an interesting one as it presents a deadlock free approach to the concept of locking.

Now, if you’ve studied computer science, you know that the presentation of a critical section to your application (in our case, that is the code accessing our locked data) - inherently presents the possibility of a deadlock.

So why does the guys at Redis claim that their approach is deadlock free?

Well, because the concept “deadlock free” refers to the process of obtaining and releasing a lock holistically.

It’s true that if a specific client acquired a lock, other clients trying to consume that data are blocked.

However - if they are following the Redlock algorithm, that lock is guaranteed to eventually be released.



In case a lock operation was successful, it has a timeout which will expire even in case the locking client crashed and haven’t gracefully released the locked resource.





Sample Context Interaction Implementation 

Here is some some python sample code, demonstrating how one process writes to the context, while another process tries to read from the context:





Rather simple, right? 

Please notice that our request to acquire a lock might be denied at a given point in time. It is then the responsibility of the lock requesting client to perform a retry.
Some Redlock implementations have that retry mechanism embedded:



Excellent. We’ve now learned how to safely access our distributed cache with concurrent consumers trying to use / modify it.

Lets see another most interesting use-case.


Interprocess Communication With Redis
	Assuming we desire our process to communicate with the “outer world”, they require access to an interprocess communication mechanism.

Redis has a very simple and efficient Publisher / Subscriber interprocess communication abstraction built into it.

Utilizing this abstraction, we can decouple our processes from any knowledge about “Who” produces or consumes any specific message.

A process, then, is only responsible to process messages arriving at his “gate”.



Let’s see how such a mechanism can be implemented with Redis.

Sample Publisher / Subscriber Mechanism Implementation
	Lets first observe how data is being published on a specific channel:



Now, lets implement a client consuming messages from this channel:



Simple, right?

Let’s see a client which consumes information from several channels:



And to conclude this demonstration, lets see a client which consumes information from several channels, but this time specifying the channels of interest as a pattern.

Again, here is the publisher code, publishing to three different channels on every loop iteration:



And here is the subscribing client, but this time subscribed to channels which name complies to the pattern specified:






In Conclusion
	We began this discussion with a point about context, and why it’s important that our context has a discrete value at any point in time.

We understood how the concept of a context is desired to be translated to in distributed systems.

At this point we examined how Redis allows us to “solve” the distributed context “problem” using the Redlock algorithm.

Once we’ve examined some code samples we moved on to examine an additional popular use-case for Redis - The Publisher / Subscriber messaging model implementation.

Again, we examined the concept and demonstrated with code. 

Please notice that I’ve intentionally left out any DevOps related discussion about clustering / high availability and fault-tolerance. I intend to write about these topics in a separate article.

Thank you for joining me,
Kobi.
