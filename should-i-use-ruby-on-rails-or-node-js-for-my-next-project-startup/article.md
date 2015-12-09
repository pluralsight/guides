First things first, I am not a Ruby on Rails veteran. I am fairly new to the framework, but have dabbled with Ruby before and understand it. Take this post with a grain of salt, this isn't a debate or article putting down X language or X framework.

I see Ruby on Rails and Node.js compared so much and this question being repeatedly asked in every crevice of the Internet I thought I would take a stab at clearing up any confusion about making a choice.

## The numbers

Firstly, lets look at the numbers for Ruby on Rails and Node.js.

Using the ever-handing [Modulecounts](http://modulecounts.com/) site which scrapes module sites for counts on a daily basis and can graph changes over time, we can easily compare the numbers between Rails and Node.js.

![ruby-on-rails-node-js-module-counts-graph-comparison](https://s3.amazonaws.com/static.written.com/ruby-on-rails-node-js-module-counts-graph-comparison1415641049.png)

We can quite clearly see that Node.js is leading the race for number of modules in a relatively short amount of time. Does this mean much? Not really. It either means people see value in Node.js and want to make modules for it (highly plausible) or creating modules for it is a lot easier than creating a Ruby GEM (also highly plausible).

All these numbers prove is that both Ruby on Rails and Node.js are pretty close in terms of popularity. You have to take into account just how many of these modules are unique and used by people. In reality you won't ever use more than a handful of modules in your Rails and Node.js projects, the number really means nothing and doesn't determine if a framework or language is good or bad.

## Apples to oranges
Comparing Ruby on Rails (a framework built on Ruby) to Node.js is like comparing apples to oranges. Node.js is an application runtime environment that allows you to write server-side applications in Javascript.

Node.js is not a framework nor a new language, the common misconception of Node.js being a framework is a popular one, same goes for calling it a language. A better comparison would perhaps be comparing Ruby on Rails to ExpressJS (a popular framework for Node.js), but even then, it's still not a true comparison.

## Opinionated vs Non-opinionated
Ruby on Rails is an opinionated Framework that makes you adhere to its way of doing things out-of-the-box. From the routing system to ActiveRecord, Rails assumes you build your apps in a particular way.

Node.js is completely unopinionated, out-of-the-box it can do basic tasks, but you'll have to write some code and install some modules if you want to match the out-of-the-box feature-set of Ruby on Rails like; interacting with a database, implementing MVC or an advanced routing system. Node.js assumes nothing and gives you the bare minimum from a fresh install. Ruby on Rails gives you a lot from a fresh install (generators included), got it?

## Speed of development
When it comes to speed of development, no matter how fast you are with Javascript and Node.js, Ruby on Rails has the game locked down when it comes to development speed. You can develop a CRUD application complete with database migrations in just a few commands via the command line using generators. Rails purists don't use generators, but there is no denying they save time.

Node.js on the other hand if you're building a web application is a little more involved, you need to find the modules, include them and then follow along with the instructions for integrating them with something like Express (also a module install).

The way Node works is that it makes you string together different components to build an application, this gives you flexibility but it takes considerably more time considering you need to download a module for practically everything.

## The Learning Curve
Any Ruby on Rails devout will tell you that learning Rails is not hard. They'll point you to the numerous tutorials and courses out there for learning Ruby on Rails, the Rails for Zombies course is a well known free course. However the flaw with most tutorials I've seen is they make you jump straight into Rails, but don't touch upon the Ruby aspect.

While you might not need to know Ruby to write _rails generate model User_ in your command line, understanding things like arrays, objects, classes, models, variables, Ruby-level methods and other things are very important. Can you truly teach a framework without first teaching the language it is built on?

The learning curve for Node.js is considerably less than Rails. This isn't a deal-breaker for people who like learning new things, but it can be a deterrent for those easily turned away in frustration. The beautiful thing about Node is that it is accessible to not only developers from backgrounds such as C or Java, it is also easy for front-end developers to pick up.

When you write a Node.js application, you're writing and learning Javascript. And while some things in Node might not port over to client-side Javascript, you're using Javascript variables, object methods, objects, arrays and understanding the language not just a framework as you learn.

## Finding Talent

To me one of the major factors when deciding whether to use X language or X framework for your next project or web startup is: am I going to be able to find talented X developers?

The major benefit of finding Node developers is that Node.js has reached a point where most front-end developers as well as back-end developers have used it in some form. Being able to hire a front-end developer who also knows Node means you're getting extra value for money and if you're a lean startup, money definitely matters.

That's not to say that finding Rails developers is hard, but generally because Ruby is a language not known by the mainstream all too well, it can be hard to find a Ruby on Rails developer with knowledge of both the underlying Ruby language and framework itself. The cost of hiring Rails developers is also a lot higher because they're in-demand and hard to come across.

## Conclusion

There is no definitive answer. Use whatever you are comfortable with choosing, especially if you're building a website or application and you're going to be the only developer. If you're a lean startup looking for the best language to iterate and develop quickly, it all comes down to preference and who you want to hire.

Both Rails and Node can achieve the same results, I would argue that Rails is perfect for situations where you need to move quickly (prototyping, functional CRUD app in the space of an hour) but a Node.js skeleton app or Git repository with some pre-grouped Node modules can be just as fast as well.

Having said that, Node.js is becoming a rather in-demand skill for a developer to have in 2014 and presumably even more-so in 2015\. I think eventually as frameworks like Express & Sails mature, we'll see the speed barrier taken down and differences between Rails and Node being reduced to almost zero.

When it comes to building your application, as long as you understand what is going on, the language or framework is irrelevant. It's the execution that matters, so go out there and build something.

