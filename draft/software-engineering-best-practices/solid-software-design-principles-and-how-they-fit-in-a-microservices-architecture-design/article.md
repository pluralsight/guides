Introduction
SOLID Software Design Principles - Brief History
The Single Responsibility Principle
The Open / Closed Principle
The Liskov Substitution Principle
The Interface Segregation Principle
The Dependency Inversion Principle
Microservices Design Principles - Todays Hype
Independent Deployment
Modularity
Communication
Size
Why and How It All Fits Together
A “One by one” approach - each service on it’s own
SOLID Software Design Principles Implementation
A “Holistic” approach - a system of (micro) services
SOLID Software Design Principles Implementation
A “Bridged” approach
SOLID Software Design Principles Implementation
In Conclusion

Introduction
	Design principles are good. They can guide us when making important decisions.
Relying on proven and mature “advice”, when designing our software, is helpful - in particular when considering the following facts:
The domain of computer-science  is rather “new” to our world.
As such, new technologies, development languages, runtimes and paradigms are rather common.
Thus, the “book” of computer-science is not yet (and might not be) complete. Chapters are being added to it constantly, as real-world requirements arise and the software industry evolves to meet them.
Relying on “best practices” when observing a newly discovered territory will prove beneficial (at least statistically).
Many markets are very competitive. Bringing your product to the shelve can have both a tight budget and time constraints.
Under such circumstances, it is desirables to have as little “open” questions as possible.
Some questions have been asked plenty of times and we can rely on proven answers to them.
Writing great software is difficult. There are many simple things you can do that are simply not correct. Acknowledging this fact is actually a driving-force towards development of better ways of doing things.
Unlike the previous bullet, here we are not talking about answers to known questions, but rather on the manner in which we approach answering our remaining open questions.
This article is about the “SOLID” software design principles in general and how they fit the sub-domain of Microservices, in particular.
I hope you join this read. Let’s get started!
SOLID Software Design Principles - Brief History
	A little after object oriented software design became popular, at the 90’s (of the 20th century), it also became clear that object oriented design is not protected from the “merits” of any human written content - It can be poorly written.
Huge classes and methods, code and / or configuration duplication, doing too many things in a single code block and much more - were still possible. And old habits are easy to get back to.
All I am trying to say here is that the mere paradigm into object oriented software design, did not solve the “mess” we were in.

It’s like todays “hype” over agile software development. Some teams, who were forced to work under the SCRUM model, are non-enthusiastically participants of daily meetings (which they consider a waste of time), and simply want to get back to their chair and work (as they think is most productive). Their approach towards planning meetings is the same.

Please don’t misinterpret me. I have nothing (in particular) against SCRUM.
I am merely stating that any paradigm / significant change in one's way of work should be adopted by that person - adopted wisely, with an understanding and commitment to the paradigm.
Ok, so we understand that it was possible to write “bad” software prior to the emergence of object oriented software design, and we also understand that it was possible to write “bad” software after the emergence of object oriented software design.
So … why bother, then?!
Well, apparently, it was easier to “bring order” and guidance to the world of object oriented software design, due to the abstractions it presented. Objects, their methods and inheritance trees, presented additional abstractions to programming languages (such as interfaces as first-class citizens) - abstractions which required caution and order when handled.
As you will shortly see, most principles covered in this article relate to recommendations about a specific piece of code (e.g. it’s responsibility, how many logical things it should perform, what it should expose to the outside world, how many times can it be replicated), with regards to an observed idea.
These recommendations (along with others) were presented by Robert C. Martin, and became known as the SOLID principles. Each letter in the word “SOLID” stands for a principle:
Single Responsibility
Open/Closed
Liskov Substitution
Interface Segregation
Dependency Inversion
While even full compliance with these principals cannot promise that your software is bullet proof - these principles were and are demonstrated again and again in successful projects in the object oriented software design world.
It even works for me :-).
Lets perform an overview if the SOLID principles in the context of their creation, first - the context of object oriented software design. Once we’ve done that - we will be able to observe these principles in the recently emerging paradigm of microservices.

The Single Responsibility Principle
The single responsibility principle states that a class should have a single reason to change.
Many people have added “their own understanding” of this principle, which translates to “a class should only perform a single task”. 
While this addition is most beneficial, it is actually a side-effect of the “true” meaning:
Classes should be highly coupled internally.
A change to a specific area of the class should affect all dependant classes. 
Now, while intuitively, this might sound like a bad thing, it is actually a driving force towards “cutting” our classes to small pieces - based on their functional responsibility in our system.
Doing so allows us to isolate a specific “area of change” in our software, according to the dependencies of the class being changed. And if we “dissect” our code properly, we will mostly discover that each small class has less dependencies than the previous “larger” class.

The Open / Closed Principle
	The “Open / Closed” principle, states that software entities should be open for extension but closed for modification.
Yeah, I know. A bit confusing at first.
Let’s first clarify what entities are.
In the world of object oriented software design, an entity is a class, module or a function. Some object oriented programming languages vary in abstraction terms (e.g. some do not contain the module abstraction), but in general they all have something in common:
They all allow us to define instances of an object, which has a unique state (unique, even if only by the address in memory it is stored at) and a common behaviour, which is shared by all other object instances.
The entities we are referring to are the code abstractions we’ve used to implement our objects with (wether its classes for an object template or functions in which we’ve written a specific behaviour of a given object).
Let’s look at an example, which violates the principle:

The Car class, has three accelerate methods - one for each car model.
In case we’ve introduced an additional car model to our system, we would have to modify the Car class itself!

That means that when extending our system, we performed an undesired modification.
Let’s look at a design which does not violate the open / closed principle:

In this design, we can see that by using an interface, and passing on the responsibility for behaviour implementation to the inheriting classes, we’ve “closed” our system for modifications, while keeping it “open” for extension (i.e. the addition of a new car model).  

The Liskov Substitution Principle
	The Liskov substitution principle states that if objects of type T are replaced with objects of type S, when S is a subtype of T, in a program - the programs characteristics should not change (e.g. correctness).
While that sounds very intuitive to you, today, you need to understand that some programming languages actually allow a subtype to modify its super-type behaviour!
So this “extension” principle is basically a warning about avoiding that pitfall.

The Interface Segregation Principle
	This principle sounds so simple, yet is so powerful:
An object should only depend on interfaces he requires and should not be enforced to implement any method (or property) it doesn’t require.
Have you ever encountered two classes which inherit the same interface, yet one (or even both) of them do not use one or more of the methods in that interface?
Sure you have. We all did.
Please take a look at the following example:

What we see here is the interface IMatrix which defines matrix operations.
We see that the interface contains the method GetInverseMatrix().
However, calculating the inverse of a matrix is only relevant to regular matrices. Why should this behaviour be described in a general interface and not a separate interface, such as IRegularMatrix? No reason, really. Let’s do it:

“Breaking down” interface described behaviours allows us to improve our codes readability, testability and maintainability. 

The Dependency Inversion Principle
For dependency inversion I will start with quoting the principle:
Higher-level modules should not depend on low-level modules. Both should depend on abstractions.
Abstractions should not depend on details. Details should depend on abstractions.
Interesting, right?
Again, it seems rather intuitive, nowadays, as we got used to “connect” our components using interfaces.
However - many beginner developers make this mistake again and again, as (apparently) our human mind tends to think hierarchically.
The following “bad” example is not that uncommon:

See the”problem” there?
Our high-level class, Car, is tightly coupled to the Engine class. The Engine class, in-turn, is tightly coupled with th Gear class.
We’ve created a tightly coupled system. Every lower-level component is being used directly by the level above it.
How can we solve this situation? Well … rather simply, using interfaces between components.

By extracting the “contract” between each dependant entities, we’ve decoupled each entity from its former dependencies. A contract is now being shared between each entity and its dependency.
Microservices Design Principles - Todays Hype
	Nowadays, it seems everyone is talking about microservices. Many claim that their software platform is a microservices platform and (most of) the rest claim that they are on their way there.
What are they all talking about? And most of all, is there anything we’ve learned which we can apply in this new domain? I think so.
Let’s first explain the core principles of a microservices software architecture.
As with any abstract, whether new or not, highly talked about paradigm / technology or concept - you might ask different people to explain it to you and get different answers.
However - most answers will relate to the following ideas and characteristics, which I will describe here.
Independent Deployment
To begin with, a microservice should be independently deployable. You software system can be deployed as a whole, but it is not mandatory. Services (microservice) can be added (and removed) from the system as it is running. A Microservice should be deployed with it’s dependencies, if any (e.g. database, 3rd-Party components).
Modularity
Each microservice is a unique piece of code, running in its separate process, possibly hosted on a separate machine than other microservices. Being a part of a bigger “puzzle”, which serves the business goals - it is responsible to provide a single “piece” or “functionality” to the system.
Communication
Microservices, when required, should communicate with the “outside world” (e.g. other microservices) using a well defined, standard and light-weight communication mechanism.
I’ve seen (and written) systems were services expose their endpoints through HTTP/REST only (with Json) and systems in which microservice communicate through queues only. And a mix of these, as well. Whichever way you choose, maintaining “order” with regard to communication standards is going to be crucial to your success.
Size
I kept it for last, though you might have expected me to talk about the “size” first,  as the name “microservice” suggests.
The size of a microservice is something which is controversial. I will provide here several opinions and mention my own. Further on, when adding the SOLID principles to the discussion, you will have the tools to make your own opinion.
The first opinion is that a microservice should be as “large” as it needs in order to perform the task he was designed for - but not “larger”.
However - some system designs might require a large number of microservices, when adhering to this recommendation. That translates into a large number of processes, which have their overhead.
Now, if you are working on a system that has a requirement to be able to run as an AIO solution, you might be limited by an overly granular “break down” of your system into processes.
Just something to keep in mind.
A Second opinion is that the size of microservices is related to … your team sizes.
According to this opinion, you should build microservices in a size such that a single development team should be able to maintain it.
Now, while there might be short-term management benefits to this approach, I disagree with it and prefer the first option as I believe in keeping software design clean of knowledge about the “outside world”.
I therefore recommend the first approach - make your microservice as large as they need to be in order to perform the task at hand. No larger. 

Why and How It All Fits Together
	At this point, you are familiar with the SOLID principles for object oriented software design.
You also have an idea about what a microservices design of a software system looks like.
But how do these design paradigms relate to each other?
Well, they do not compare well. However, they stack well and please allow me to elaborate.
To begin with, when writing any concrete microservice, you can still enjoy the benefits of the mature and elaborate knowledge and experience gathered in the domain of object oriented software design. That knowledge is not thrown away. It can be reused :-).
Microservices can be developed using any paradigm (e.g. functional programming, object oriented, event driven) or flavour you choose, that's true.
Being practical, most modern programming languages and frameworks are object oriented - making this article of interest to many of you.
Several ways to approach the consolidation of the two ideas (i.e. object oriented software design and microservices software architecture):
A “One by one” approach - each service on it’s own
	With this approach, we are developing each service separately. Completely. It will even have its own source control repository.
With this approach, there are, as always, pros and cons.
One con, for example, is that sharing “common” utility functions is not allowed. And if, for some reason, you cannot extract that functionality into its own microservice - you will have functionality duplicated across your system.
However, this approach encourages choosing the right tool for the task at hand, as repositories are unbound from one another. Each repository can be of any programming language, build technology and 3rd party dependencies.
SOLID Software Design Principles Implementation
With this extreme approach, were each service is completely isolated, even at the level of it’s codebase, we will apply the SOLID software design principles per service.
No shared codebase is desired, so we will treat each microservice as a whole application.
This means that it will have it’s own interfaces, class hierarchy and dependencies.
This approach is interesting, though it can certainly lead to logic duplication in services.
For example, each microservice will maintain its implementation for working with the interprocess messaging system.
In some scenarios this will be desirable (e.g. working with a lightweight version of an http client if possible), but in most it will probably not. 
A “Holistic” approach - a system of (micro) services
Taken to the other “extreme”, a microservice is only thought of as a piece of the larger picture.
Now, please don’t kill the messenger! I know most of you heard that this is not “microservices way”. I know. But it is a “way”, and has some pros (and cons) to it.
For example, one pro (which is also a con) is the ability to maintain a single repository for all services.
SOLID Software Design Principles Implementation
Code sharing and reusability is much easier with this approach.
For example, you could share your previously duplicated messaging code, among services (as long as they are written with the same programming language).
Is it a “good” thing? Arguable. Will you encounter this approach in systems? Probably.
The largest benefit of this approach is that it is easier to maintain a single repository with a single build system and be able to “compile” and test your latest code together.
In turn, the largest pitfall of this approach is that versioning a single service becomes a nightmare.
In case you’ve made a change to a single service, you still need to compile and build the entire system and there is always the risk that you’ve “broken” something.
The scope in which we can maintain and enforce the SOLID principles when taking this approach is maximized when looking at the system from the “outside” but unfortunately, is being reduced as we dive in to specific services.
One example - when looking at the system as a “whole”, we might have violated the Open / Closed principle. Each change to a service might cause us to change code / rebuild our system.
A “Bridged” approach
Life, as life goes, will present difficulties. No single approach you take, will guard you from tough choices.
Choosing the “by the book” approach, with each microservice managed in it’s own repository, having its own build and versioning process will certainly have benefits. But also an overhead (e.g. repository management).
Then again, choosing the “holistic” approach, where all services are managed and built from the same codebase - will have less overhead in some aspects (e.g. build time, code duplication), it will also present “problems” when considering others (e.g. service versioning).
Can we think of a “nice place in the middle” where the two approaches can converge?
I think that in most cases, we can.
For example, how about presenting an abstraction of a “microservice group”?
Now, a group of microservices might be defined by the technologies and dependencies shared by the services.
Or by the logical aspects they are managing.
The “grouping” of microservices should not be strict, but rather agile. This means that if we’ve come to understand that grouping services according to specific parameters doesn’t work well for us - we can always change our grouping definition.
Using this approach, we could look “holistically” at a group of microservices - and yet have control over this groups versioning, build process, dependencies and codebase management.
SOLID Software Design Principles Implementation
No surprises here :-). We will eventually only be able to “keep an eye” on services which are bound together (e.g. by contracts, shared repository).
However - introducing the concept of a “service group”, we could take control of the tradeoffs between the two extremist approaches I’ve previously presented.
Within the bounds of a service group, we could share dependencies, build process and most importantly - a codebase which respects the SOLID software design principles.
In Conclusion
	In this short journey, I’ve tried to do three things (yeah, I know, it violates the single responsibility principle).
The first thing was to explain the SOLID software design principles. In particular, I’ve tried to present them in a favorable manner. Why? Because they are proven beneficial time after time.
Then, I’ve presented the new kid on the block - the concept of a microservice.
I began with explaining that the terms are not comparable, but rather (optionally) composable - you can combine the two ideas into your system design.
Mostly, I’ve tried to emphasize the fact that working with microservices is interesting but can present an overhead in some aspects. An overhead, in particular when implemented in a “strict academical” manner.
Then, I’ve taken the “other side” and presented the other extreme point of view.
To conclude this article, I’ve presented with an idea of how to bridge the two point of views.
Fortunately, I’ve been around long enough to experience both “extreme” implementations and came to understand that for practical reasons, agility and an open mind is the key to success.
Therefor, whether or not you take my proposed idea for bridging the two point of views, or decide to go in a completely different way - I wish you success and remind you to always check where your overhead is - then, optimize :-).
Thank you very much for baring with me.
Feel free to drop me a word at the comments section.
Thank you,
Kobi.
