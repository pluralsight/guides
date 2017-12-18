
Introduction
Common Use Cases
Shifting from Push to Pull - Preventing Client Overload
Reliable Messaging Systems
Load Balancing
Parallel Processing
In Conclusion



 
Introduction
	There are common situations in which your software, even when properly written and debugged, will face bottlenecks.

These bottlenecks will mostly be related to the hosting machine and its limitations.
In some cases we will be limited by the capabilities of our CPU. In other cases it will be the machines disk I/O capabilities. RAM can also become a bottleneck - even for today's monstrous machines.

In this article, we will try to examine several of these situations and suggest a queue based solution to them.

This article will be an “overview” article and will not be specific to any technology (though relevant technologies will be mentioned).

If you feel like a specific topic is of specific interest to you, please drop me a word in the comments section and I will try to provide a follow-up article.

Let’s get started!
 
Common Use Cases
	The use cases we will examine are the following:

1.	Managing the consumption rate of data per client, through shifting from push to pull.
2.	Making unreliable messaging between software components reliable.
3.	Introducing load-balancing to our system, using queues and multi-instance (of software components)  deployment.
4.	Utilizing resources in parallel, when possible, in order to process more data per unit of time.

There are probably more use cases where queues can and are being used. This article is about minimizing bottlenecks and will focus on relevant use cases.
 
Shifting from Push to Pull - Preventing Client Overload
	We all make the mistake of writing a piece of code to which we send requests to.


 

Writing that code is not the problem, of course :-).

Not considering the scenario where this piece of code has to process more requests than it can is.

 

In cases where there is a direct connection between a message producer and a  message consumer, several things can happen when the producer can produce more messages than the consumer can consume:
In case the producer waits for the previous message to be fully consumed, following messages will be produced with delay.

 

In case the producer doesn’t wait for the previous message to be fully consumed, following messages will be produced on-time, but will be consumed with delay.

 

In both scenarios, a bottleneck will be generated in-process. This is bad.

This could lead to a bloat in RAM usage on either sides, where the messages are waiting to be consumed.

 

The bottlenecked process uses so much RAM that it slows down as paging takes place and possibly eventually blocks the entire host machine.

 

So up to this point was the bearer of bad news. Now the good news - these situations can easily be optimized using a … queue.

Please take a look at the following proposed architecture:

 

In the diagram above, you can see that the message producer(s) and message consumer(s) are not communicating directly.

The message producer produces messages into a queue as soon as they are ready.
We can say that the producer “pushes” messages into the queue.

 
The message consumer then consumes messages from the queue as soon as he is able to.
We can say that the consumer “pulls” messages from the queue.

 

Let’s see the status of our mentioned problems..

To begin with, messages will no longer be delayed from being produced (assuming our queue is capable enough, which is a fair assumption, using technologies such as MSMQ or Apache Kafka).

 
Then, our consumer is still able to consume just as many messages as before but control over rate of message consumption was passed to it.

 

Related to the two but it does so without “bloating” and risk of out of memory exception, as pending messages are pending in the queue and not in our processes RAM. We will see how we can increase message consumption rate a bit later in this article.

Seems nice, right? Outsourcing our message rate of message production / consumption bottlenecks to a third party technology, thus making our software platform more reliable (i.e. reduced risk of out of memory exceptions).

Lets see what else we can achieve using queues.

 
Reliable Messaging Systems
	The previous topic, while being related to message delivery reliability, did not address the reliability concern directly. 

It helped us achieve better system reliability as it taught us how to handle a scenario of a faster message production rate than message consumption rate. 

Let’s take on the challenge of reliable messaging in our system, this time explicitly.

It is very common to see systems passing messages directly between processes.

 

I will not completely dismiss that there are use cases where such a scenario may be desirable in your system.

However, I will say that for common messaging requirements - such software design is flawed.

To begin with, it requires the producing service to be familiar with the consuming service.

That means that our producing service has either a configuration or a service discovery mechanism.

 

We will actually have to write code that behaves according to its configuration, manages endpoints (“discovers” new endpoints, load balancing traffic among them) and basically has additional infrastructure modules built into it.

I claim that these concern should remain in the realm of DevOps, and not software design.
Service discovery, load balancing, dynamic system scaling and reliable messaging should be taken care of by the infrastructure managed by our DevOps team!.

I believe that a software engineer should focus at the algorithmic problem he is trying to solve, and simply declare the messages he wants to consume and have the infrastructure deliver these messages to his code.

 

In the same way, when his code has something to say to the world (i.e. a message), it will simply produce a relevant message to the infrastructure and move on with its algorithmic problem solving.

 

Using a decent queue technology (such as Apache Kafka, which is much more than a queue actually), will allow us to do just that - separate the concern of reliable messaging in our system from our code base - and outsource that concern into a proven, mature technology which focuses on doing just that. Having done that, we’ve also created a healthy integration point between our software development team and our DevOps team.

 

Now - I don’t mean that its a healthy integration point from social reasons, but from the product development life cycle aspect.

Our DevOps team will deploy our software to their environment and provide feedback on messaging issues, bottlenecks and / or unacceptable delays in message relaying even before our version reached the QA team! Possibly even as part of automated build test on an environment “closer to real-life” than the developers environment.

Early detection of problems = money saved.

Modern queue technologies has message delivery acknowledgment built into them in a transparent manner, meaning that the entire process of making sure a message reached its destination is completely handled by it.

 

To summarize this topic: 

I’ve proposed to use a queue as a mean to interprocess communication among system services, in particular in order to increase message delivery reliability. Having a mature technology taking care of our messaging requirements is not bad.
It helps us focus on the algorithmic problem at hand.

I’ve also reminded the fact that such a platform could be completely outsourced to the DevOps team, leaving our developers with the sole task of solving the algorithmic problem they are working on solving.

Then, I mentioned the fact that a queue based messaging infrastructure is an enabler to a scalable, fault-tolerant and load balanced software system.
 

 
Load Balancing
Load-balancing is definitely something that can be achieved using queue technologies (again, please take a look at Apache Kafka).

Without using a queue, assuming you introduced an additional service instance to your system, you will have to introduce a load balancing mechanism as well:

 

This load balancer will have to take care of:

First - “knowing” which service instances are available. This will usually be done either by service configuration or by a service “discovery” mechanism.
Both these solutions require additional development and are error prone.
Not to say that these approaches are not valid - just that they require careful attention to specific details (e.g. service state - did a service crash?).

 

Then - it will need to be able to evenly spread messages among available service instances.
That means that he will usually need to “remember” how many messages each service got (unless it is randomly sending messages to instances).

 

At last, the load balancer will be the only entity which can take care of message delivery acknowledgments as only he knows which service should acknowledge each message.

 

And these three tasks I’ve just mentioned need be done per each service we want more than one instance running of, at a given time!

Ok, what about an alternative? :-)

What about using a … message queue ...   

I propose using a message queue per each message type.

This means that all messages of the same type are produced into the same queue.

 

In turn, all messages of the same type are also consumed from the same queue (obviously, as they are only stored there).

 

All your system services need know of is the queue connection details and relevant message queue name(s)

 

That means that introducing additional service instances to your system, in order to tackle a bottleneck will be as simple as spawning a new service instance (assuming you do not store service state in-memory). That’s it! As the new service is establishing a connection with the queue, the queue will distribute messages among the instances for you (per your configuration).

 

Rather simple, right? We’ve increased our systems scale, and increased our throughput using a queue as our load-balancer.

Now, please notice that this load balancing we’ve achieved can also be ignorant of its consumers capabilities to process data as they are pulling data from it at their own rate!
 
Parallel Processing
	The last use case I will discuss in this article is the case of bottlenecks which are caused because of “large” computations.

In the case of a “large” computation that cannot be broken down into smaller, separate parts, we will probably have to scale our system by bringing online additional services which can compute such “large” computations.

 

However, there are computations which can be broken down into separate, “smaller” computations that can be performed separately.

 

With such computations, as I’ve just mentioned, our system could be optimized by processing several “smaller” computations separately, in parallel, and then combining the results of these computations to the final result for the entire “large” computation.

 

Please keep in mind that parallelism is usually limited. You cannot just “break” a problem into as many “smaller” problems as you desire and just throw these at your CPU.

Each CPU has a physical thread count which should be taken under consideration when performing a computation in parallel.

In case you don’t use all available CPU physical threads, you are probably under utilizing your resources.

But be careful - in case your software requires to perform more calculations, in parallel, than there are physical threads in your machine, you will pay the overhead caused by context switching.

But how will a queue help us here?

Again, just as mentioned before, when discussing separation of concerns between the developers and the DevOps team - it can help us write code that is agnostic of the environment it is hosted on.

Please consider the following example:
Our system is handling “large” computations.

Service A receives input messages which he knows how to “break down” into three discrete “smaller” computations, which are performed by Service B, Service C and Service D.
 
 

Our system runs on a machine with eight available physical threads.

That means that optimally we would like to have eight computations done in parallel.

Our software developers need only concern themselves with writing the application in a way that allows performing the three discrete computations in parallel.

That means that each different computation should have a dedicated message queue and service which reads messages from this queue and processes them.

That’s all from the developers point of view!

Now, on the DevOps side, we would like to make sure that two things happen:

1.	All message queues are created.
2.	The system is deployed with the correct degree of parallelism.

For the example above, as each message can be broken down to three “smaller” messages, we would like to have three service instances brought up (one for Service A, one for Service B and one for Service C) and be ready to receive messages per each “large” message. So we have three physical threads plus one physical thread = four physical threads per “large” message.

Having eight physical threads at our disposal, we can safely bring up an additional instance of service A, an additional instance for Service B, one more for Service C and finally an instance of Service D.

Our system will then look like this:

 

Now, what happens if need to scale down and be hosted on a machine with lower resources? Not a software problem … Not a single line of code will be modified. Instances can be taken down (in the diagram below we are bringing down four instances - reverting to the above deployment).

 

What about scaling out into additional machines? Not a software problem again. Not a single line of code will be written! Bring up additional services and point them at your message queue. That's all there is to it, realy. 

 
 
That means that our developers need concern themselves with “breaking” down problems into discrete computation units and make sure each computation type has a unique message queue.

If you follow these guidelines, your application will be able to perform computations in parallel, to the extent of hardware resources (i.e. physical threads). 

 
In Conclusion
There are many possible causes for system bottlenecks.
In this article I tried to address several of the most common ones, as is my experience in the software world.

While the causes of such bottlenecks might differ - they do share a common thing. All of them can be optimized to remove (or at least optimize to the best of our hardware resources) these bottlenecks using message queues.

When combined, the practices described in this article can assist in building a reliable, more robust software system with support for changing scale and optimized to perform with a maximal utilization of hardware resources.

Not merely our (less lines of) code will benefit from using a message queue, but also our product development life-cycle.

Scaling our system can become the domain of our DevOps team. And it needn't be more complicated than spawning additional service instances. 

It can really be that simple :-)

I hope you’ve enjoyed this overview article. You are more than welcome to write me any request you have from me to elaborate on anything I’ve written here.

Kobi.
