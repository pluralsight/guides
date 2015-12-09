Sometimes when you get puzzled by what Rails is doing, you really ought to understand what Ruby is doing. For example, take this simple code to get an attribute value:

``` ruby
# return value of some_attribute and foobar
def some_attribute_foobar
  "#{some_attribute} and foobar"
end
```

Beginners are often stumped by why this code does not set an attribute value:

``` ruby
# change the value of some_attribute to foobar
def change_some_attribute
  # why doesn't the next line set the some_attribute value to "foobar"?
  some_attribute = "foobar" save!
end
```


What’s going on? In the first method, some_attribute is actually a method call which gets the attribute value of the record. This works in Rails ActiveRecord due to the Ruby feature of [method_missing](http://www.ruby-doc.org/core-2.1.3/BasicObject.html) which
allows some code to run when a method is called that does not exist. In the second method, a local variable called some_attribute is getting assigned. There is no call to method_missing, as this is a variable assignment! The correct code should have been:

``` ruby
# change the value of some_attribute to foobar
def change_some_attribute
  self.some_attribute = "foobar"
  save!
end
```

In this case, we’re calling the method some_attribute= on the model instance, and we get the expected result of assigning an attribute value.

## Enums

For those not familiar with enums:

> An **enum** type is a special data type that enables for a variable to be a set of predefined constants. The variable must be equal to one of the values that have been predefined for it.

Enums, introduced in Rails 4.1, are a place where a lot of Ruby magic happens! It’s critical to understand Ruby well in order to understand how to use enums effectively.  Let’s suppose we have this simple example, copied over from the [Rails docs](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html):

``` ruby
class Conversation < ActiveRecord::Base
  enum status: [ :active, :archived ]
end

# conversation.update! status: 0
conversation.active!
conversation.active? # => true
conversation.status # => "active"

# conversation.update! status:1
conversation.archived!
conversation.archived? # => true
conversation.status # => "archived"

# conversation.update! status: 1

conversation.status = "archived"

# conversation.update! status: nil
conversation.status = nil
conversation.status.nil? # => true
  conversation.status # => nil
```


So what’s going on in terms of Ruby meta-programming? For all the enum values declared for Conversation, methods are created in the following forms. Let’s use the model Conversation, column “status”, and the enum “active” for this example:


| Method | Description |
| ------ | ----------- |
| self.status | Returns enum string value (not symbol, and not integer db value) |
| self.status= | Set the status to corresponding enum integer value using either a string, symbol, or integer. If you use an invalid value, you get an ArgumentError. String/symbol is converted to corresponding integer value. |
| self.active! | Sets the status enum to “active”. This syntax is a bit confusing in that you don’t see the attribute you’re assigning! ArgumentError if invalid enum. |
| self.active? | equivalent to (self.status = “active”), and *not* equivalent to (=self.status = :active=) due to symbols not being equal to strings! |
| Conversation.active | equivalent to Conversation.where(status: "active"). Again, it’s a bit confusing not to see the column being queried. |
| Conversation.statuses | Mapping of symbols to ordinal values { "active" \=> 0, "archived" \=> 1 }, of typeHashWithIndifferentAccess, meaning you can use symbols or strings |


### Default Values for Enums

As the docs say, it’s a good idea to use the default value from the database declaration, like:
``` ruby
create_table :conversations do |t|
  t.column :status, :integer, default: 0, null: false
end
```


More specifically, consider using the first declared
status (enum db value zero) be the default and to not allow null values. I’ve found that when I’ve allowed null values in enums, it makes my code MORE complicated. This is an example of the
[Null Object Pattern](http://robots.thoughtbot.com/rails-refactoring-example-introduce-null-object). Including nulls in your data and checking for them will make your life MORE difficult! Instead, you can actually set an enum value for “I don’t
know”, and make such the first value, which is an index of zero so that you can set it as the database column default.

### Queries on Enums

The docs say:

> In rare circumstances you might need to access the mapping directly. The mappings are exposed through a class method with the pluralized attribute name

``` ruby
Conversation.statuses # => { "active" => 0, "archived" => 1 }
```

This is not rare! This is critical! For example, suppose you want to query where the status is not “archived”: You might be tempted to think that
Rails will be smart enough to figure out that

``` ruby
Conversation.where("status <> ?", "archived")
```

 Rails is actually not able to detect that the ? is for status and that it is an enum. So you have to use this syntax:

``` ruby
Conversation.where("status
<> ?", Conversation.statuses[:archived])
```

You might be tempted to think that this would work:

``` ruby
Conversation.where.not(status: :archived)
```

However, this throws an ArgumentError. Rails wants an integer and not a symbol, and
symbol does not define to_i. What’s worse is this one:

``` ruby
Conversation.where.not(status: "archived")
```

The problem is that ActiveRecord sees that the enum column is of type integer and calls #to_i on the value, so archived.to_i gets
converted to zero. In fact, all your enums will get converted to zero! And if you use the value of the enum attribute on an ActiveRecord instance (say a Conversation object), then you’re using a string value! If you’re curious what the Rails source is,
then take a look here: [ActiveRecord::Type::Integer](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/type/integer.rb). Here’s a guaranteed to be broken bit of code:

``` ruby
# my_conversation.status is a String!
Conversation.where.not(status:my_conversation.status)
```

You’d think that Rails would be clever enough to see that the key maps to an enum and then check if the comparison value is a String, and then it would not call to_i on the String! Instead, we are effectively running this
code:

``` ruby
Conversation.where.not(status: 0)
```

An acceptable alternative to the last code example would be:

``` ruby
Conversation.where.not(Conersation.statuses[my_conversation.status])
```

If you left out the not, you could also do:
``` ruby
Conversation.send(my_conversation.status)
```

I would like to simply do the following, however; all of which DO NOT work.:
``` ruby
Conversation.where(status: my_conversation.status)
Conversation.where(status: :archived)
Conversation.where(status: "archived")
```

### Pluck vs Map with Enums

Here’s another subtle issue with enums. Should the two lines of code below yield the same result or different results?

``` ruby
statuses_with_map = Conversation.select(:status).where.not(status: nil).distinct.map(&amp;:status)
statuses_with_pluck = Conversation.distinct.where.not(status:nil).pluck(:status)
```

It’s worth experimenting with this in the [Pry console](http://www.railsonmaui.com/blog/2014/08/17/pry-ruby-and-fun-with-the-hash-constructor/)! In the first case, with map, you get back an Array with 2 strings:
`["active", "archived"]`. In the second case, with pluck, you get back an Array with 2 integers: [0, 1]. What’s going on here? In the code where map calls the status method on each Conversation record, the status method converts the database integer value
into the corresponding String value! In the other code that uses :pluck, you get back the raw database value. It’s arguable whether or not Rails should intelligently transform this value into the string equivalent, since that is what is done in other
uses of ActiveRecord. Changing this would be problematic, as there could be code that depends on getting back the numerical value.

### find_or_initialize_by, oh my!!!

Let’s suppose we have this persisted in the database:
``` ruby
Conversation {
  :id => 18,
  :user => 25
  :status => "archived" (1 in database)
}
```

And then we do a find_or_initialize_by:
``` ruby
[47] (pry) main: 0> conversation = Conversation.find_or_initialize_by(user:25, status: "archived")
  Conversation Load (4.6ms) SELECT "conversations".* FROM "conversations"
    WHERE "conversations"."user_id" = 25
      AND "conversations"."status" = 0 LIMIT 1
#<Conversation:> {
           :id => nil,
      :user_id => 25,
       :status => "archived"
}
```

We got nil for :id, meaning that we’re creating a new record. Wouldn’t you expect to find the existing record? Well, maybe not given the way that ActiveRecord.where works,
per the above discussion. Next, the status on the new record is created with “archived”, which is value 1\. Hmmm….If you look closely above, the query uses

``` ruby
AND "conversations"."status" = 0
```

Let’s look at another example:
``` ruby
Conversation {
  :id => 19,
  :user => 26,
  :status => "active" (0 in database)
}
```

And then we do a find_or_initialize_by:
``` ruby
[47] (pry) main: 0> conversation = Conversation.find_or_initialize_by(user: 26, status: "active")
  Conversation Load (4.6ms) SELECT "conversations".* FROM "conversations"
    WHERE "conversations"."user_id" = 26
      AND "conversations"."status" = 0 LIMIT 1
#<Conversation:> {
         :id => 19,
    :user_id => 26,
     :status => "active"
}
```

Wow! Is this a source of subtle bugs and some serious yak shaving? Note, the above applies equally to ActiveRecord.find_or_create_by. It turns out that the Rails methods that allow creation of a record
via a Hash of attributes will convert the enum strings to the proper integer values, but this is not case when querying!

### Rails Default Accessors For Setting Attributes

You may find it useful to know which Rails methods call the “Default Accessor” versus just going to the database directly. That makes all the difference in terms of whether or not you can/should use the string values for enums. The key thing is that that
“Uses Default Accessor” means that string enums get converted to the correct database integer values.

| Method | Uses Default Accessor (converts string enums to integers!) |
| ------ | ----------- |
| attribute= | Yes |
| write_attribute | No |
| update_attribute | Yes |
| attributes= | Yes |
| update | Yes |
| update_column | No |
| update_columns | No |
| Conversation::update | Yes |
| Conversation::update_all | No |


#### For more information on this topic, see

1. [Different Ways to Set Attributes in ActiveRecord](http://www.davidverhasselt.com/set-attributes-in-activerecord/) by [@DavidVerhasselt](https://twitter.com/DavidVerhasselt).
2. [Official API of ActiveRecord::Base](http://api.rubyonrails.org/classes/ActiveRecord/Base.html)
3. [Official Readme of Active Record – Object-relational mapping put on rails](http://api.rubyonrails.org/files/activerecord/README_rdoc.html).

While these don’t mention Rails enums, it’s critical to understand that enums create default accessors that do the mapping to and from Strings. So when you call these methods, the default accessors are used:

``` ruby
conversation.status = "archived"
conversation.status = 1
puts conversation.status # prints "archived"
```

Thus, it is important to keep this in mind when those default accessors are used per the above table.

### Deep Dive: Enum Source

If you look at the Rails [source code for ActiveRecord::Enum](https://github.com/rails/rails/blob/877ea784e4cd0d539bdfbd15839ae3d28169b156/activerecord/lib/active_record/enum.rb#L82), you can see this at line 91, for the setter of the enum (I added some comments):

``` ruby
_enum_methods_module.module_eval do
  # def status=(value) self[:status] = statuses[value] end
  define_method("#{name}=") { |value|
    if enum_values.has_key?(value) || value.blank?
      # set the db value to the integer value for the enum
      self[name] = enum_values[value]
    elsif enum_values.has_value?(value) # values contains the integer
      self[name] = value
    else
      # enum_values did not have the key or value passed
      raise ArgumentError, "'#{value}' is not a valid #{name}"
    end
  }
```

From this definition, you see that both of these work:

``` ruby
conversation.status = "active"
conversation.status = 0
```

Here’s the definition for the getter, which I’ve edited a bit for illustrative purposes:
``` ruby
# def status() statuses.key self[:status] end
define_method(name) do
  db_value = self[name] # such as 0 or 1
  enum_values.key(db_value) # the key value, like "archived" for db_value 1
end
```

### Recommendations to the Rails Core Team

In response to this issue, I submitted this github issue: [Rails where query should see value is an enum and convert a string #17226](https://github.com/rails/rails/issues/17226)

1. @Bounga and @rafaelfranca on Github suggest that we can’t automatically convert enum string values in queries. I think that is true for converting cases of a ? or a named param, but I suspect that a quick map lookup to see that the attribute is an enum, and a string is passed, and then converting the string value to an integer is the right thing to do for 2 reasons:

  **1\.** This is the sort of “magic” that I expect from Rails.

  **2\.** Existing methods find_or_initialize_by and find_or_create_by will result in obscure bugs when string params are passed for enums. However, it’s worth considering if all default accessor methods (setters) should be consistently be called for purposes of passing values in a map to such methods. I would venture that Rails enums are some Rails provided magic, and thus they should have a special case. If this shouldn’t go into Rails, then possibly a gem extension could provide a method like Model.where_with_enum which would convert a String into the proper numerical value for the enum. I’m not a huge fan of the generated Model scopes for enums, as I like to see what database field is being queried against.
2. Aside from putting automatic conversion of the enum hash attributes, I recommend we change the automatic conversion of Strings to integers to use the stricter Integer(some_string) rather than some_string.to_i. The difference is considerable, String#to_i is extremely permissive. Try it in a console. With the to_i method, any number characters at the beginning of the String are converted to an Integer. If the first character is not a number, 0 is returned, which is almost certainly a default enum value. Thus, this simple change would make it extremely clear when an enum string is improperly used. I would guess that this would make some existing code crash, but in all circumstances for a valid reason. As to whether this change should be done for all integer attributes is a different discussion, as that could have backwards compatibility ramifications.This change would require changing the tests in [ActiveRecord::ConnectionAdapters::TypesTest](https://github.com/rails/rails/blob/master/activerecord/test/cases/types_test.rb). For example, this test: `assert_equal 0, type.type_cast_from_user('bad')` would change to throw an exception, unless the cases are restricted to using Integer.new() for enums. It is inconsistent that some type conversions throw exceptions, such as converting a symbol to an integer. Whether or not they should is much larger issue. In the case of enums, I definitely believe that proper enum string value should not silently convert to zero every time.

## Conclusion

I hope this article has convinced you that it’s worth understanding Ruby as much as it is to understand Rails. Additionally, the new Enum feature in 4.1 requires some careful attention!

