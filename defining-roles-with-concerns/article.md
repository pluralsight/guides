Ruby on Rails strives for productivity by focusing on what is important and leaving the not-so-important things to be dealt with convention over configuration. We have all heard this every now and again and it rings a bell every time I hear it, because the more I dive into the language the more I understand how powerful this is.

The dynamic behavior of Ruby fits well with Rails and brings a lot of flexibility to the developers, but with great power comes great responsibility. I was recently working on a code that started to get coupled too fast, and my first approach was to try to improve it with inheritance.

Here is the scenario. I had a method that recursively iterated over a collection. At first, the stop rule was clear and things worked out perfectly well, the code was clean and short, and everybody was happy.

Then, chaos ensued. Taking the drama apart, what (always) happens is that the scope changed and another stop rule knocked on my door. At this point, I decided it was good enough to extract the first to a method and create another with the new requirement, and I could call the recursive function passing which rule that was going to stop the loop.

Guess what happens next: another rule. And another. And another. And another.

I could not simply continue bloating the class with methods, and I am sure you already imagined something like this sneering at me from the code:

```ruby
case <restriction type>
when <criteria 1>
  method1(params)
when <criteria 2>
  method2(params)
when <criteria 3>
  method3(params)
else
  default_method(params)
end
```

Neat, huh? Well, not so much. My first instinct was to extract those methods to their own classes. It would not solve the problem of the _case-when_ statement unless I had a base class for them and used duck type to inject the stop rule in the method.

Things started to make sense. The duck type would save me from having to instantiate classes inside my main class, which would couple it with the criteria and completely ignore the single responsibility principle. My class would know too much.

And still, things were awry. I could not fully grasp what was behind the goose bumps I was feeling. The code worked, but I was not happy with it, I knew that a few weeks later I would not easily remember what all those classes were for or what they were supposed to do. If someone else had to implement a new rule, would they know what method they should override? How long would it take to discover it? How could I simplify the tests that were repeated everywhere?

Sandi Metz came to the rescue. I was, by a lucky chance, reading her book _Practical Object-Oriented Design in Ruby_ (which I **highly recommend**), and there the secret solution was unveiled to me. I understood why that code was bothering me so much and changed my frame of mind to point it in the right direction.

Here is what I had:


|-- base_rule.rb

|--- max_hops_rule.rb

|--- max_cost_rule.rb

|--- no_repetition_rule.rb

|--- …


To make things more visual, let's write some code. We have a `Node` class, which is a pair of origin-destination (let's say, A->B) values. We also have a list of nodes and must find paths from origin to destination, and a rule (or a set of rules, but to simplify the code let's stick with just one) will be applied to stop the loop. For example, we can limit the path to 30 hops at most, or to at least 5.

First, the object that will represent one step in the whole path:

```ruby
class Node
  attr_reader :origin, :destination

  def initialize(origin, destination)
    @origin = origin
    @destination = destination
  end

  def to_s
    "#{@origin}->#{@destination}"
  end
end
```

The `.to_s` method is just to simplify the visualization of what is going on. Instead of looking at an object, we can print it and see 'A->B' instead of `<Node:0x007fe2aa857250 @origin="A", @destination="B">`, for example. It helps a lot in the tests too.

Next, a rule and its base class:

```ruby
class BaseRule
  def stop?(_)
    true
  end
end

class MaxHopRule < BaseRule
  def initialize(max: 5)
    @max = max
  end

  def stop?(list)
    list.count == @max
  end
end
```
Last, but not least, our main class, the one that will iterate through the nodes and find paths from the origin to the desired destination. If you are not very familiar with ruby, don't fret yourself. The idea here is just a loop that follows these simple rules:

1. Take all the nodes in the list that starts in the origin;
2. Iterate on this list checking if any of them ends in the destination. If yes, save the path up to that node plus the current node on the result list. 
3. For each node on this list, recursively call the method again, but change the `origin` parameter to be the current node's `destination` (if no stop rule prevents it from doing so).

Here's the code:

```ruby
class Route
  attr_reader :nodes, :results

  def initialize(nodes)
    @nodes = nodes
    @results = []
  end

  def find(origin, destination, base_rule)
    navigate(origin, destination, base_rule)

    results
  end

  def navigate(origin, destination, stop_rule, breadcrumb = [])
    valid_nodes(origin: origin).each do |node|
      current_list = breadcrumb + [node]
      results << current_list if node.destination == destination

      unless stop_rule.stop?(current_list)
        navigate(node.destination, destination, stop_rule, current_list)
      end
    end
  end

  private

  def valid_nodes(origin:)
    nodes.select { |node| node.origin == origin }
  end
end
```

Inheritance, while efficient in some cases, is not the answer to everything. It is hard to see what must be rewritten on the child classes if the relationship among them are there just to force the existence of a common method. I was looking to the problem at the wrong angle:

> I should have looked at it from the point of view of the class with the recursive method, not from the classes that were going to stop the loop.

Those rules I was injecting in my class had no meaning outside the main class context, they existed only to interact with it. To fix everything, all I needed to realize was that those rules were playing a role, and that all that mattered to me while designing my application was how they were going to interact with my `find` method.

The rules classes were **restrictors**. They interacted with the `Route` and required it to stop according to a set of rules they were designed to enforce. The need for a common method to be used as a duck type is nothing more than an interface, and although this is a common practice in languages such as Java or C#, they are not very clear on Ruby on Rails.

But fear not. The fact that we do not explicitly declare an interface does not mean it is not there. When a set of classes respond to the same method, either by inheritance or by any other solution you can concoct, they are responding to an interface.

The problem with the implicit interface is that it is difficult to future developers (or to your future self, which happens very often) to see what needs to be implemented, to understand that there is an interface in the first place.

Up to now, we decided to move from an inheritance solution to one based on roles. We had a bunch of classes inheriting from `BaseRule`, whose only purpose was to provide a common method to be called on our main class, and we will cut out this inheritance chain.

By accepting that moving from a traditional inheritance to a role based relationship is a good move (it is not always the case), we can enjoy some benefits that come with it:

* Any other class can act as the role, no matter what it is or from whom it inherits. That's because we left the _is-a_ relationship to a more _behaves-like_ one.
* We can focus on the messages that are moving between the classes instead of the classes themselves.
* Classes that act as a role can have their own agenda, they are not strictly bound to any other class or chain of inheritance.

This does not mean that inheritance is bad, neither that the inclusion of modules is a superior solution to every possible case. It is just a new possibility that can even be combined with the other, but beware to do not add too much complexity just for the sake of doing it.

In this case, it worked like a charm because it clearly showed to me the responsibility of each role, and I knew at first sight what they were supposed to do (it was previously hidden into the depths of the traditional inheritance). Any new developer can immediately understand what is going on now. In my first code things were not so outrageously obvious (albeit it was not impossible to grasp it).

The roles are small and pluggable. This allows them to be easily usable in other contexts as long as the contract is respected, and a change in one role hardly affects the others because they are unique and isolated. If well designed, even a big system refactor will not break any of them, which means they _invite_ changes in the system, and this is the main purpose of OO in the first place. As **Sandi Metz** said,

_"These small objects have a single responsibility and specify their own behavior. They are transparent; it’s easy to understand the code and it’s clear what will happen if it changes. Also, the composed object’s independence from the hierarchy means that it inherits very little code and so is generally immune from suffering side effects as a result of changes to classes above it in the hierarchy."_

And the thing that I like the most about `concerns` is how we can document the relationship between the classes on the tests. Our only truly up-to-date documentation is the code itself, and the tests are a great place to understand what things were designed to do.

Thinking about the roles, there are some tests that will certainly be repeated all over again for each role class. For instance, we could want to be sure that all of them respond to a certain method (like the `stop?` method).

Instead of repeating ourselves, we can instead create shared examples that will run for all of the roles. Every class that includes our module will automatically gain these tests with not a single repeated line. 
```ruby
shared_examples_for 'a restrictor' do
  it { should respond_to(:stop?) }
end
```

More than that, the code that will include those shared tests on the roles are also beautifully telling us the class behaves like a role:

```ruby
it_behaves_like 'a restrictor'
```

_A tip: when creating a file with shared examples, do not use the suffix spec on the file name, or the tests will run twice. I use to create them on the spec/support folder_

This is simply perfect. Just by taking a peek at the top of the test file we instantly know it behaves like a restrictor, which implies it agreed to an implicit interface and will, thus, respond to a set of expected methods. To remove the inheritance, all we need is a very small change. First, remove the `BaseRule` class. Then, we can create our concern:

```ruby
module Restrictor
  extend ActiveSupport::Concern

  included do
    def stop?(list)
      true
    end
  end
end
```

Finally, let's take a last peek at the MaxHopRule?

```ruby
class MaxHopRule
  include Restrictor

  def initialize(max: 5)
    @max = max
  end

  def stop?(list)
    list.count == @max
  end
end

```


