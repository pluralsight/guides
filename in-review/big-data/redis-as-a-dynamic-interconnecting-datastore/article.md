# Introduction
> “Context and memory play powerful roles in all the truly great meals in one's life.”
Anthony Bourdain, Chef

Yes, I know. He (Anthony) is talking about meals, and we are talking about software development. But there is slight similarity. Sometimes our data, passing through our software parts, goes through specific transformations, getting passed around according to a very specific algorithm. The output of that data will eventually become our output, or “meal”, so to speak.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/af24f02e-70c0-46eb-a7ff-8fd101caba67.png)


The output of each computation step could either be the final output of our processing or the input for the next step in processing.

Our current processing step is referred to as our **state**. Any data we have computed to this point is our **state data**.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/cd99100a-2560-496c-b72b-9f82ee13ad25.png)


Now, the state data might go through processing, in the current processing state, with different parameters each time (i.e. current date and time, user id). These parameters are referred to as **context**. It is very common that a message either contains it’s context or has a hint about how to resolve the context. If the context is being passed inside the message, then there is no real “resolving” step.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/184260a8-dd75-43e4-a014-dc4719a12306.png)


Each processing state might require access to the context of the data being passed to it, in order to fulfill its designed purpose.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4a1def24-a51c-41d2-a5f9-ec4e89632feb.png)


So context resolution is the process in which a running software component receives the context relevant to the computation it is about to perform.

In some common cases, such as web applications, that context could be the session established per user &mdash; when the user logged into our system &mdash; or it could contain any metadata about the information being passed to us.

Working with software systems these days is quite different from how it was several years ago. Various software components might **change our context as time passes.** These components need access to a shared resource to which they can simultaneously refer and consume (read / write) the context they so need.

[Redis](https://redis.io/) provides such capabilities.

# So … What is Redis?

Redis, originally created by Salvatore Sanfilippo, was named “Redis” to reflect its original purpose to provide “**Re**mote **Di**ctionary **S**erver”. It was initially released on [GitHub](https://github.com/antirez/redis/) in 2010 and has undergone many new releases since.


In this guide, I will provide a birds-eye view of Redis, as it is today &mdash; in my opinion, it is more than a remote dictionary server.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8b297d50-a12b-43c6-a849-383aebd65189.png)


You, the Redis consumer, will be hosting a **Redis client** in your application.
That client will communicate any request given to it to the **Redis server** (or servers, when deployed as a cluster). This communication will be performed over a simple and efficient protocol, over **TCP**.

The requests we pass on to Redis will be mostly CRUD (Create, Read, Update, Delete) operations in the form `<Key, Object>`, where Object can take on the form of any [supported data type](https://redis.io/topics/data-types-intro), such as a string, a list, an ordered list or a set.

For example, the following information was persisted in Redis for demonstration (I am using the Open source [Redis commander user interface hosted on docker](https://github.com/tenstartups/redis-commander-docker)):


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0e27c4de-9459-41ec-b7aa-e5b27c22ee8e.png)

In the image above, you can see a simple Key, Value pair. The next screenshot shows a list of values associated with the same key. 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0ac22781-7746-4362-98d6-c8d7debdda89.png)

We can optionally define an **expiration time for any key** we store in Redis. This expiration time will be accurate to the millisecond (version >= 2.6).
Expired keys (and their values) will be deleted automatically once they’ve expired.

It’s important to note that what makes Redis very interesting as a centralized Cache / Datastore / Synchronized data provider is that it is **blazing fast**, mostly because **all data is persisted in-memory**. By default, Redis will also persist all data to non-volatile storage, a setting that can be configured, and it also supports replication and high-availability &mdash; though this article won't discuss those features in much depth.

Let’s observe some use cases and see some code to better understand the technology.



# Using Redis As a Synchronized Cache Mechanism

Redis is commonly used as a distributed cache. Having more than one process in our application, Redis can be a “central point of reference” to all processes (optionally across more than one machine).

Having a central, single representation of our systems data allows us to safely implement our logic, “knowing” that at any given **point in** time per **input message**, all processes “looking” at a specific piece of information (e.g. context) will “see” the **same information**.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6e9ca4bd-4d1c-4757-85e8-e9c36d3dc117.png)


In the diagram above, we can see two processes which each received input message #1 at the exact same time. Both processes are requesting the message context from Redis and will receive the exact same context!

A different situation could also occur. It might be the case that the output of a single process is the input for another. In such a case, the times `t1` and `t2` will be different (`t2 > t1`), resulting in an optionally different context for each process.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/cd517721-7155-4f1e-9b30-f2a2a247630a.png)


For example, assume our computation begins as a user logs into our system. Before the user performs any additional operations, we **already “know”** something about that user (i.e. name, login location, and historical activities in our system).

Some of that information might be used across our computation, probably by different processes (hosted on different machines).

In such a case, we might do wisely to cache that context information in-memory and have it provided quickly to any consuming process.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/71b963dd-f650-4a14-8ba0-beec974478c6.png)


That is very helpful indeed. As long as we can make sure that our data is properly synchronized.

Remember that point about having a single representation of our context at a given point in time? Well, it appears that making sure this is true is not a simple a task when dealing with [concurrent](https://en.wikipedia.org/wiki/Concurrent_computing) systems. Multiple processes might access the same data at the same time. Some might try to modify it.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/82208be5-3da3-4c2b-9fb2-a1cfa202a120.png)


Let's see how Redis allows us to deal with concurrent data consumption.
 

# A Distributed Locking Mechanism at Your Disposal

Redis supports and promotes the uses of the **[Redlock algorithm](https://redis.io/topics/distlock)**, in order to implement distributed locking of resources. The algorithm is an interesting one as it presents a **deadlock free** approach to the concept of locking.

Now, if you’ve studied computer science, you might know that the presentation of a critical section to your application &mdash; in our case, that is the code accessing our locked data &mdash; inherently presents the possibility of a deadlock.

So how can the Redis team claim that their approach is deadlock free?

Because the concept “deadlock free” refers to the process of obtaining **and releasing** a lock holistically. 

It’s true that if a specific client acquired a lock, other clients trying to consume that data get blocked.

However, using the Redlock algorithm, that lock is **guaranteed to eventually be released**, making the algorithm deadlock free.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d41543d6-29db-4914-a61b-fe466c561950.png)


Every successful lock operation has a timeout that expires even if the locking client crashes without gracefully releasing the locked resource.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b71dc752-6204-4b3f-88f3-da94358052d1.png)


# Sample Context Interaction Implementation 

Here is some Python sample code, demonstrating how one process writes to the context while another process tries to read from the context:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2e6d25a6-a2a4-4c88-a175-8ed24932cefd.png)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7f243d64-628e-4251-8d5b-27bb7905646b.png)

Simple, right? 

Please notice that our request to acquire a lock can be denied at a given point in time. It is then the responsibility of the lock requesting client to perform a retry.

Some Redlock implementations have that retry mechanism embedded:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7c2eb0ee-98c7-4695-a725-41498758dba3.png)


Excellent. We have now learned how to safely access our distributed cache with concurrent consumers trying to access or modify it.

Let's now look at another interesting Redis use-case.

# Interprocess Communication With Redis
	
Suppose we would like our process to communicate with the “outer world”. We would need access to an **interprocess communication mechanism**.

Redis has very simple and efficient [Publisher/Subscriber](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) interprocess communication abstraction [built into it](https://redis.io/topics/pubsub) already.

With this abstraction, we can decouple our processes from any knowledge about “Who” produces or consumes any specific message.

A process, then, is only responsible for processing messages arriving at its “gate”.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/e2930164-6769-468d-9b89-87f960916ff9.png)


Let’s see how one would implement such a mechanism using Redis.

# Sample Publisher/Subscriber Mechanism Implementation

First, observe how we can publish data on a specific channel:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fdd95166-1024-47b8-b472-a31d728c60df.png)


Next, check out how a client can consume messages from this channel:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5e5abc84-b71d-4f6d-943e-805ecf15ee99.png)


Simple, right?

The next screenshot demonstrates how a client consumes information from several channels:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2cf5954d-afd4-4a59-8b77-5946085fb74b.png)


And, to conclude this demonstration, let's see a client that consumes information from several channels, but this time we will specify the channels of interest as a pattern.

Again, here is the publisher code, publishing to three different channels on every loop iteration:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/97f39ddb-335a-4fed-b020-8397ee23c403.png)


And here is the subscribing client, but this time subscribed to channels which name complies to the pattern specified:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/03f5852f-9a1d-4e9e-8676-73f407005940.png)


# In Conclusion

To summarize, we began this discussion with a point about context, and why it is important that our context has a discrete value or state at any point in time.

Then, we covered how the concept of a context features in distributed systems, particularly when it comes to synchronized caches. As part of this, we examined locking and how Redis “solves” the distributed context problem using the Redlock algorithm. 

Finally, we examined another popular use-case for Redis &mdash; the Publisher/Subscriber messaging model implementation. In covering this, we examined the concept of publishing and demonstrated with Python code. 

*Please notice that I intentionally left out any DevOps-related discussion about clustering/high availability and fault-tolerance. I intend to write about these topics in a separate article!*

___

I hope you found this Redis tutorial informative and engaging. Please leave any comments and feedback in the discussion section below, and don't forget to add this guide to your favorites. Thank you for joining me,

Kobi.
