I have always suspected
[OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) to be
overused, and
[this article](https://codewords.hackerschool.com/issues/one/why-are-objects-so-hard-to-debug)
does an excellent job articulating my suspicions. I've been developing a large
framework during my day job for over a year in Python without any
object-oriented code. So, I wanted to mention a few additional benefits I
realized with the procedural development methodology.

## 1. Unit Testing

This particular project is so easy to unit-test I cannot help myself from
writing tests each time a new feature is added.  It's no surprise that project
is the best tested application I've ever written.  The lack of objects and
class hierarchies frees the tests from boiler-plate setup, tear down, and
complicated object mocking.  The tests primarily use
[dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
because all functionality is described with function arguments instead of
state in objects.

It's also easier to see that a function contains too much logic. This is not
always the case with classes. A class can contain many different methods. Each
method might be very concise but the sum of all those methods, i.e. the class,
needs to be tested. Those tests often require lots of complicated setup just to
build up the proper situation to test a single method.  This is typically not
the case with stand-alone functions.

## 2. Developer Documentation 

My project has two main goals: provide an importable application framework to
build specific oil and gas business logic into existing applications and
provide an executable to automate many common reservoir simulation pre and
post-processing tasks.

These two goals are closely related in terms of the implementation, but the
documentation for each goal is very different and requires a different writing
style.  Developers need detailed documentation with code snippets whereas users
of the executable don't have a background in software development.  Thus, users
of the executable expect more prose and a high-level discussion of the
functionality.

Luckily the developer documentation is easy to write.  The procedural style
allows most of the functionality to stand on its own. There's no need to
describe the relationship between different objects and how to integrate them.
The documentation is simply a documented
[API](https://en.wikipedia.org/wiki/Application_programming_interface) along
with a bunch of examples. Developers can typically learn about each function in
the API as they need it. This is different than a lot of object-oriented
documentation which requires a lot of background knowledge before being able to
use an object.

## 3. Refactoring

Refactoring is extremely simple for this project. There are no internal
dependencies amongst the majority of the functionality.  Each function can be
refactored by itself.  So, refactoring takes place a lot more often since it
can be accomplished in small increments.

Refactoring is easier in part due to benefit number 1.  Unit tests exist to
catch regressions faster.

### Warning

There are definitely times when
[OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) is the right
choice, but it's not always the right tool for the job.  The OOP style is
overused because it's often the only tool many new developers have at their
disposal. This is especially true for developers who know only languages such
as [Java](https://en.wikipedia.org/wiki/Java), which requires the OOP paradigm
in all applications.  As the
[saying](https://en.wikipedia.org/wiki/Law_of_the_instrument) goes:

    I suppose it is tempting, if the only tool you have is a hammer, to treat
    everything as if it were a nail.
