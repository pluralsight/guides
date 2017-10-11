This is a quick guide showing you necessary steps you should take to make `rspec` work on `irb`.

Lets start by loading rspec core and expectations.

```
require 'rspec/core' # this is what really runs your tests
require 'rspec/expectations' # readable syntax for checking properties of your code 
```

We then include the matchers modules.

```
include RSpec::Matchers # rspec will NOT automatically include this for you if you are using irb

```

With this we  can now test out our expectations:
```ruby
>> expect(1).to eq(1)
=> true
```

Whooohoo! You are good to go.. Not really, try this out.
```
2.3.0 :003 > array_hashes = [{lol: nil}]
 => [{:lol=>nil}] 
2.3.0 :005 > include RSpec::Matchers
 => Object 
2.3.0 :006 > expect(array_hashes).to include(have_key(:lol))

```

You will encounter an error as shown below:
```
TypeError: wrong argument type RSpec::Matchers::BuiltIn::Has (expected Module)
        from (irb):6:in `include'
        from (irb):6
        from /usr/local/rvm/rubies/ruby-2.3.0/bin/irb:11:in `<main>'
```

If you read closely into the error it expects a Module as an argument, this is because when using irb we are running in the main ruby object. This object has another method called `include` that has presidence over rspec's `include` method

To work around this, we can `prepend RSpec::Matcher`s onto main's `singleton` class:

```
$ irb
irb(main):001:0> require 'rspec/expectations'
=> true
irb(main):002:0> singleton_class.prepend RSpec::Matchers
=> #<Class:#<Object:0x007ff08a8d6620>>
irb(main):003:0> array_hashes = [{lol: nil}]
=> [{:lol=>nil}]
irb(main):004:0> expect(array_hashes).to include(have_key(:lol))
=> true
```

We encountered this problem while doing a mob session at [AgileVentures](https://agileventures.org) and [raised the issue](https://github.com/rspec/rspec-expectations/issues/1018) on the rspec expectations github page where we got this hack.

