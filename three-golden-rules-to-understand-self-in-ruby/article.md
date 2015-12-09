Becoming “self aware” can be a challenging topic for ruby programmers new to object oriented programming (OOP). It’s often difficult to master a concept, like a `class` or an `object`, that has no 3D, real­-world counterpart. This difficulty is only intensified when you introduce yet another abstract idea, like that of `self`, on top of the first. In this article, we’ll explore a few simple rules that will clarify both the definition and the relationship of `self` in the context of its use in ruby classes. If you're new to ruby OOP, your life is about to get so much easier.

#### **The Golden Three Rules of Self**

1. Use `self` when setting/getting instance attributes inside a class definition.
2. Use `self` to denote a method within the class definition as a class method.
3. Use `self` to reference the calling object within an instance method definition.

Keep in mind that these rules don’t apply to every single use case of `self`, but this will take care of 9 out of 10 applications. The first pattern in the rules is that `self` only has a meaning in the context of a Ruby class. Therefore, it can only be called within an instance method or a class method. Let’s look at an example.

``` ruby
module Singable
  def sing
    "I'm #{self.name}, and I'm singing!"
  end
end

class Person
  include Singable
  attr_accessor:name

  def initialize(name)
    @name=name
  end
end

#========irbsession========

> Shane = Person.new("Shane")
  => #<Person:0x14ak3@name="Shane">
> Shane.sing
  =>"I'm Shane, and I'm singing!"
> sing
  => undefined local variable or method 'sing' for main:Object (NameError)
> Singable.sing
  => undefined method 'sing' for Singable:Module (NoMethodError)

```


In the example above, we’ve created both a simple class to describe a `Person` and a simple module, which gives that person the ability to `sing`. While we could have simply added the `sing` method to the `Person` class, we’ll instead assume that we wanted to give that singing ability to another class later on. For this reason, it’s wise to [DRY](http://http://c2.com/cgi/wiki?DontRepeatYourself) out the code by putting that behavior into a module and simply including it in all applicable classes.

The `sing` method above references the `name` attribute of the calling object and in the [irb (interactive ruby)](http://en.wikipedia.org/wiki/Interactive_Ruby_Shell) session, the calling object is a `Person` named "Shane". This is an example of **Golden Rule 3: Use self to reference the calling object within an instance method definition**. Since the `Singable` module was included within the `Person` class, any method defined in the module will be inherited by the class as if it was _originally defined in the class_. In this case, it’s being called from the Shane instance of the `Person` class, so it returns that `Person` object.

This is also a great example of **Golden Rule 1: Use self when getting/setting attributes within a class definition**. The `sing` method uses [string interpolation](http://rubymonk.com/learning/books/1-ruby-primer/chapters/5-strings/lessons/31-string-basics) to insert the `name` attribute of the calling object, Shane. Since the method is called by a `Person` object, `self` references that specific instance and the `sing` method reads the `name` attribute.

That’s all nice and good, but what exactly is `self`, anyway? I’m glad you asked, because now we can explore a little bit more about “referencing the calling object”. Take a look below.

``` ruby
class Person
  # ... all that other stuff from above ...

  def existential_crisis
    self
  end
end

# ======== irb session ========

> Shane = Person.new("Shane")
  => #<Person:0x34b29@name="Shane">
> s = Shane.existential_crisis
  => #<Person:0x34b29@name="Shane">
> Shane.name
  =>"Shane"
> s.name
  =>"Shane"
```

As you can see from the second example, “referencing the calling object” simply means that the returned value of the expression is equal to the object that made the call. In the case of this brief irb session, `self` is equal to the `Person` object Shane, because Shane called the `existential_crisis` method. We can also prove that they are equal by calling the `name` attributes of both instances and seeing that they both return the `Person` object’s `name` as “Shane”.

We’re almost done with our short tour of the Golden Rules. We’ve explored object attributes as well as instance methods, but there’s one last implementation of `self` that's worth looking into. Let’s keep track of how many `Person` objects we’ve created by updating the class below.

``` ruby
class Person

  include Singable
  attr_accessor :name

  @@crowd = 0

  def initialize(name)
    @name = name
    @@crowd += 1
  end

  def existential_crisis
    self
  end

  def self.population
    @@crowd
  end
end

# ======== irb session ========

> a = Person.new("Aaron")
  => #<Person:0x344h9@name="Aaron">
> b = Person.new("Bill")
  => #<Person:0x984k2@name="Bill">
> c = Person.new("Charlie")
  => #<Person:0x8fj83@name="Charlie">
> Person.population
  => 3
> a.population
  => undefined method `population' for #<Person:0x344h9@name="Aaron">
     (NoMethodError)
```

Some programmers who are new to object oriented programming have difficulty understanding the difference between a class method and an instance method. I won’t go into detail about those differences, but you can learn more about the difference [here](http://www.railstips.org/blog/archives/2009/05/11/class-and-instance-methods-in-ruby/). Instead, I’ll summarize by saying that instance methods are behaviors that can only be operated by an instance of a particular class, and class methods are behaviors that can only be operated by the class object itself.

Remember earlier, when we called the `Shane.existential_crisis` method? In that example, we employed an _instance_ of the `Person` class, Shane, to call the method. So, to refer to `existential_crisis` as an instance method makes sense, right?

In much the same way, we can call methods on any class itself without “initializing” a new instance of that class in ruby. We’ve actually been doing it this whole time without realizing it! When we call `Person.new()`, ruby interprets that as the `initialize()` method, which is a special case because we don’t have to use the `self` keyword.

To make things apparent though, we first created the `population()` class method, and then we updated the `initialize()` method to increment the `@@crowd` [class variable](http://www.tutorialspoint.com/ruby/ruby_variables.htm) every time a new `Person` is created. In irb, we created three new `Person` objects. We can clearly observe this when we execute the class method `Person.population` on the next line of the irb session. In the definition of the method, we prefix it with `self` in order to denote it as a class method, just like in **Golden Rule 2: Use self to denote a method within the class definition as a class method.** You can see that it won’t work as an instance method when we try to run the same method on the first `Person` object we created.

So that’s it! Be sure to learn The Three Golden Rules of Self, and you shouldn’t have any issues with `self` in your ruby programs any more. And if you find yourself needing a little more explanation, you can always open up a request here on [hack.hands( )](https://hackhands.com/dashboard/request/expert) to get on­-demand help from a ruby pro.

