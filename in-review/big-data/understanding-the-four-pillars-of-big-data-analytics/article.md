Contents
Understanding the Four Pillars of Big Data Analytics	2
Introduction	2
Overview	3
Common Concerns for All Four Pillars	3
Collecting - The Data backbone	4
“Online” Analytics - The Computation layer	5
Real Life Example	7
Getting Assistance From a Data Model	7
Context Resolving as a Constraint	8
Persisting - The Storage layer	9
“Offline” and Complementary “Online” Analytics - The Insights engine	10
Summary	11

 
Understanding the Four Pillars of Big Data Analytics

“Big Data is a term encompassing the use of techniques to capture, process, analyze and visualize potentially large datasets in a reasonable timeframe not accessible to standard IT technologies.” By extension, the platform, tools and software used for this purpose are collectively called “Big Data technologies” 
NESSI (2012)

Introduction

Wouldn’t it be great if we understood what all these guys and gals are talking about in the recent years? The term “Big Data” is all around us. An industry is growing around the term, providing data centered services which claim to enhance your business in some manners.
And make no mistake – I truly believe some of these claims.
It’s important to understand that even though you might have just started to hear people talk about big data – the problem domain has been around for many years. Data availability has grown vastly since the Internet became widely available. Moreover, data became very interesting to commercial companies once they understood that the content of that data can translate to business.
Universities and researchers also understand the power of data, in particular - plenty of data.
In this article, I will try to build up, step by step, the readers understanding about how big data analytics is mostly done today. 
You are more than welcome to join me for a quick read.

 
Overview

 

	Throughout this article, we will describe the “moving parts” in a typical big data analytics platform. I am saying “typical” as other approaches may exist, that will also benefit your enterprise or research. However – in my experience, the following approach is most common and capable.
For each pillar (or role) we examine, I will explain how it fits into the bigger picture, and mention the specific responsibilities this role has.
For each such role, I will mention relevant popular technologies that will serve well as an implementation fitted to the task of big data analytics.

Common Concerns for All Four Pillars

	What is the difference between any data analytics platform to a “Big data” analytics platform?
Why doesn’t the “legacy” manner of things suffice? Can’t we simply write an application that receives data via any standard application program interface (aka API), parses and ingests it and finally displays some interesting insights?

Well … We can, but to an extent. We will reach our limit, resource wise, very fast.
Any single modern machine, be it as fast as it gets, with the fastest storage and network devices money can buy today – will still have an upper limit to the amount of data it can manage in any given time.

Another concern, when working with a single machine, is that the system itself is a single point of failure. Hardware WILL fail sometime and this will mean data loss to your enterprise. We can’t have that.
Putting these concerns to terms, we understand that our desired system should be:
•	Scalable - for us to be able to overcome the data management limitations any single machine might have. 
•	Fault-tolerant - so that our system remains operational even when something unexpected happened (such as a hard drive failure or power surge).
Meeting both the requirements, will mean that our system need be:
•	Distributed - We need to be able to extend our system and add additional resources to it to raise the upper limits of its data management capabilities (Collecting, processing, storing, replicating, gathering insights and more).
As our system is built as a chain of technologies, we need make sure each link in the chain conforms to our requirements. Let’s get started.

Collecting - The Data backbone

	As expected, our journey begins with data 😊. To be more specific – with data collection and interception.
In some cases, our system will actively request for data from relevant data sources. In this case we will say that our system pulls data from these data sources.
An example for such a scenario might be a periodic query to a database, which results are being consumed.
In other cases, our system will register for data reception from data sources and will receive new data as it is available. In such a case we will say that the system receives data from these data source via a push mechanism.
An example for this scenario might be the usage of a technology such as Logstash, which we could configure to pass data from (almost any) data sources and into our systems entry point.
The technology (or technologies) we choose to use to collect and prepare data for our computational requirements is the first Pillar of our system – The Data Backbone.
The data backbone role, is the entry point in and into our system. It’s sole responsibility is to relay data to the other links in the chain that is our data analytics platform.
But let’s not make the mistake of over simplification here. While the data backbone has a single role, it is not an easy one!
I remind you that the data backbone is required to be scalable and fault-tolerant, under changing rates of incoming data – which may be a short burst of data or a constant stream of massive amounts of data.
The capabilities of our data backbone will dictate weather or not we will lose important data!
In some cases, data will be produced only once and if we don’t “catch” it in time – we will lose it.
To conclude our discussion about the data backbone, I would like to summarize our requirements from this specific role, in our big data analytics system.
•	Our data backbone, as a data collection and reception mechanism, is expected to provide data to the rest of the system – regardless of how it gets this data, actively via pull or passively via push mechanisms.
•	As data sources may vary in characteristics, our data backbone should be simple enough to integrate with. Working with standard API protocols will probably benefit your enterprise.
•	For the data backbone to be reliable, we need it to be scalable and fault-tolerant.

A data backbone creation and utilization, using the amazing Apache Kafka, can be seen here, in the following Pluralsight course: Building an Enterprise Grade Distributed Online Analytics Platform.
  	
“Online” Analytics - The Computation layer

	Once we’ve built a rock-solid data backbone, data can be streamed into the rest of our system.
As soon as new data arrives, we would probably want to observe it and determine if it is of special interest to us.
The following scenarios are possible, given a fresh dataset which just arrived through the data backbone:
•	The new dataset is logically complete and insights can be generated directly from it. 
For example – a specific event we are watching for, such as “Panic button pressed” event in an adult monitoring system.
•	The new dataset is logically complete and insights can be generated about it, when related to a context.
For example – An “Add to cart” event, on an online shopping site, when no “Payment” event happened within five minutes.
•	The new dataset is a part of a logical dataset, which wasn’t yet composed.
An example here could be that we’ve received a part of a satellite image, but would like to analyze it only once all the image parts are available.
•	The new dataset is a part of a logical dataset and insights can be generated about it, when related to a context.
A common example here is a video clip, which comprises of multiple frames with multiple packets per frame.
The role of the computation layer is to provide you with the tools to do just that – contextualize and complement any given dataset so that we can answer analytical questions.
Let’s look at the following diagram:
 

Assuming a fresh dataset arrived at our computation layer, we will possibly need to verify that it is logically complete. If it isn’t, we will probably persist it and wait until we have a logically complete dataset (hopefully, in the future).
But then again, if it is logically complete, we might want to ask analytical questions about it. In other words, we would like to perform a computation on it.
As previously mentioned, we might want to observe any logical dataset in context.
Here lies an interesting aspect of the computation layer in big data systems.
As our computation layer is a distributed system, to meet the requirements of scalability and fault-tolerance – we need to be able to synchronize it’s moving parts with a shared state.
This shared state mechanism will be a blazing fast persistence / caching technology.
Each dataset which arrives at our computation layers gate, will have to be persisted at the context providing mechanism prior to any computation. 


 
Real Life Example

For example, please consider the following use case:
We have a rather “naive” analytical question: we would like to know how many orders of a specific product were placed in each minute.
To do so, we’ve implemented an analytical question in our computation layer that does just that.
As we are expecting a massive flow of events, we’ve scaled our computation layer and we have multiple processes listening to these order events.
Now – process #1 receives an order for product a. it counts it and checks if the counter passed the threshold for insight generation. If it did – an insight is generated. Simple enough, right? wrong.
This implementation does not take into consideration the possible (and very likely) scenario were order events arrive at multiple processes simultaneously!
That means that our order event counter should be synchronized.
And that is the reason we need an external (to a computation layer process) synchronized context provider to whom all (context aware) analytical questions refer to.

Getting Assistance From a Data Model

	One last thing worth mentioning – If you’ve noticed, the first diagram in this article includes an optional entity named “data model derivation” which is linked to the computation layer as well as to the shortly reviewed storage layer.
When refereeing to a data model, in the realm of big data, we usually refer to data of interest –organized  (or structured) in a manner that is suited for analytic derivation.
For example – it might be the case that we have a data model, which the aggregation of cellular data usage, partitioned by cities and logically persisted as a decision tree. 
Given a pre-calculated data model, our big data analytics system can relate to already calculated results (or even insights) in it’s search for new insights.
It is very common that the data model is:
•	Persisted in our storage layer.
It is sometimes the case, that our data model is kept separately, persisted using a different technology than the one we use for our “online” persistence needs.
For example, it might be the case that we are using Apache Cassandra as our persistence of choice, due to it’s blazing fast writes and performance scalability capabilities – but using a different technology in order to persist large data files which we will analyze “offline”, due to its fast read performance.
•	Calculated in an “offline” manner, periodically.
This is usually due to a necessity – performing highly (time, CPU, I/O) consuming computations on large datasets is something that is … well … (time, CPU, I/O) consuming. Anything we cannot do “online” – or do not want to pay the extra cost of doing “online”, in case it is possible but requires high cost resources – might be left for a later “offline” computation.
•	Pre-loaded into the computation layer, which relates to it in two manners:
o	Reads data from it, to deduce insights.
o	Writes fresh data to its loaded representation, so it is kept “up to date”, until we are given a freshly calculated data model.

Context Resolving as a Constraint

Now, let’s pause for a moment and understand a most important aspect of data analytics, which becomes more of an issue when dealing with “big data” – There is a limit to the amount of data we can process at a given time.
It’s true – with a scalable system, we can push the limits higher, but when observing a system at a discrete point in time – limitations exist.
The limitations I am referring to, are particularly relevant under the headline “online analytics”, when the word “online” can be translated to “close enough to the data creation time”.
In turn, “close enough” can be translated to a specific period of time – specific to your business requirements.
Context resolving relies, first and foremost on inter process communication. Persisting and retrieving our datasets context metadata requires that we “reach out” and request services from an external mechanism.
To make things worse, this external mechanism is a synchronized mechanism, which means that (at least logical) locking of resources takes place!
Then – there is also the concern of context metadata persistence. Not all data can be (and by all means - shouldn’t be) stored in memory. I remind you – we live in a distributed ecosystem. We have multiple processes, running on multiple physical machines. Our local memory is … local to a physical machine.
To top that, I remind you that we are dealing with “big data”. We might have so much data at our gate that we simply cannot hold it in memory, wholly. 
So we understand that our synchronized context provider has to both synchronize and persist data.
We also understand that synchronization and persistence of data takes time.
If you look back at the title of this subject, you will notice that it refers to “online” analytics. 
Given that we are required to analyze any incoming dataset within a limited amount of time (for example, 5 seconds) – we will not be able to do so if our computation time, added to the time cost of our interactions with the context provider exceeds that amount of time. At least not in “online” manner.  
A computation layer, built using Apache Storm, and integrated with a full-blown analytics system, can be seen here.

Persisting - The Storage layer

	Though we’ve already mentioned the storage layer in our discussion of the previous “Pillars”, or layers as I usually refer to them – Let’s examine our requirements from a big data analytics compliant storage layer.
To begin with, it must be able to persist incoming data very fast – or it will become a system bottleneck, which will again – have an undesired impact on our capability to perform calculations in an “online” manner.
Making things interesting, it must be able to manage changing rates of incoming data. Preferably, we would like to be able to change the scale of our storage layer on demand.
Losing data is something non of us desires in an enterprise grade data analytics system – hence, we demand of our storage layer to be reliable.
What does our requirement for reliability translates into?
Well, basically, as we are dealing with distributed technologies – we are talking about data replication and fault-tolerance.
We would like to make sure our system can withstand an occasional failure (hardware or software), and remain functional.
We would also like to make sure that our data wasn’t corrupted or lost, in such a case.
I would like to remind you, now, that our storage layer, just as any other layer in our system, is not limited to a single technology.
It might be desirable to utilize several technologies, under the hoods, to make sure we are meeting all our enterprises requirements.

For example, let’s consider the following system:
Our system is given the following input:
•	A constant stream of stock trade records.
•	A daily summary of all stock trading information, in a file.
To analyze the short-term stock trade trends, our system analyzes a sliding window of 10 minutes. Per each incoming dataset – our computation layer resolves incoming stock trade record against it’s relevant context (e.g. transaction count for the same stock, in the last 10 minutes).
If our analytical question produced an insight, our computation layer will report it.
In any case, we expect our computation layer to update the relevant context, so that following calculations will be correct. So, we require of our storage layer to hold our context.
In turn, we will receive a massive file, daily, which we will parse, process and generate a data model from. We will do that “offline”, optionally with the same technology we used for gathering “online” analytics – though that is not mandatory.
Then, once our data model is ready, our computation layer could take advantage of it and generate insights according to the data it reflects – data which relates to a larger scope (a day) than we’ve related to before (10 minutes). 
Please notice that I said “could take advantage of” and not “should” or “would”.
There is no thumb rule here – it could very well be that the data model is used directly by your systems analysts.
If you would like to see for yourself how a storage layer can be implemented, using Apache Cassandra, take a look here.

“Offline” and Complementary “Online” Analytics - The Insights engine

	Excellent! Getting this far you already understand the very basics of big data analytics.
Most important, you understand that to be able to handle “big data” (as defined in the beginning of the article) – we require special tools (i.e. distributed technologies).
Using these tools, we will be able to implement a robust data backbone, computation and storage layer.
And that’s pretty much it, right? well, not always …

Building a distributed system with so many moving parts, requires that we integrate the technologies. This integration of technologies, is what gives us the ability to cherry pick the best technology for the task at hand – thus allowing us to raise the limits of what analytical questions we can ask in an “online” manner.
Unfortunately – it also means that our system complexity rises. Several technologies piped together, means that your developers and data analysts need know more to develop new analytical questions and integrate them into the whole system.

Ok, but that is a limitation we need to live with, right? Well, yes. but …
What if we could do better? In particular – do much better at a very low cost?

Assuming we’ve built a big data analytics system, with the knowledge we have up to this point, we now have a solid big data analytics platform which answers a pre-defined set of questions.
This system has its limitations, as previously discussed.
To begin with, its ability to answer analytical questions within a specific time frame (a time frame dictated by our requirement to answer questions in an “online” manner) is not boundless – as we’ve seen in our discussion about limitations posed by the context providing mechanism, for example.  

One way to overcome such limitations, and complement our system with the ability to answer ad-hoc analytical questions, will be to pass all our raw data into a search and analytics engine, such as elasticsearch – and maybe even put the Kibana cherry on top.

In case you are unfamiliar with elasticsearch or Kibana, I will just mention that these are two distributed, scalable and fault-tolerant technologies which work together amazingly to bring both sophisticated queries as well as dynamic dashboards – to your system.

The first, elasticsearch, is a search and analytics engine, which abstracts the usage of the most capable Lucene full text search engine – and brings forth a simple API, as well a query domain specific language (i.e. DSL).

The second, Kibana, brings the ability of data visualization to your system. By defining dashboards, time series, charts, geo spatial data, and much more – your analytical tools box grows nice 😊.
Kibana also allows direct interaction with you elasticsearch cluster, in the form of analytical queries, written with a rather simple DSL, as mentioned before.

Summary

	Throughout this short journey together, we’ve mentioned several times that tackling big data analytic problems, will require paying attention to the following:
•	Changing rate and size of incoming datasets.
o	From here we derived our requirement for system scalability.
•	The importance of data completeness and our desire not to miss data.
o	And here we derived our requirement for fault-tolerance
From these two requirements, we’ve concluded with the result that a distributed system design is required.
Breaking down the system, we’ve discussed the four pillars (or roles / layers) which together, form a common big data analytics platform, today.
We began our journey with observing the data backbone, which is responsible to provide data to the rest of the system.
Moving forward, we discussed the computation layer, which (using or not using a context) asked the actual analytical questions we had in mind (optionally against a pre-calculated data model).
At this point in time, we were introduced to the storage layer, which provides persistence services for both “online” and “offline” analytics. It stores the data model, in case we have one.
Finally, we discussed a complementary component in the form of an Insights engine, which allows us to get more out of the raw data, in case we haven’t built into the system all our analytical questions.

Thanks for joining me in this short journey of understanding the four pillars of big data analytics 😊.
