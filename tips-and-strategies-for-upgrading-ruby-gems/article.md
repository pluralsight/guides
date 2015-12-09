What are the best-practices for upgrading gems to newer versions? What sort of tips and techniques can save time and headaches?

I built this guide based on my real-world experiences over years of gem migrations, including a recent upgrade to Rails 4.1, RSpec 3.0, and Twitter Bootstrap 3.

## Why Update?

Here’s my favorite reasons for keeping gems relatively current:

1. If you work on several projects, keeping the gems and ruby version consistent makes your coding more productive as you don’t have to keep adjusting for which version is which. Web searches tend to find relatively recent versions first. It’s relatively annoying to be yak shaving issues that turn out to be “oh, that doesn’t work in that older version of Rails”.
2. Recent versions of gems will have fixes for bugs and security issues, in addition to new features. With popular open source projects, new bugs are quickly discovered and fixed.
3. Updates are much easier if you stay relatively current. I.e., it’s much easier to update from Rails 4.0 to Rails 4.1 than to go from Rails 3.0 to Rails 4.1.

That being said, recent versions can have new bugs, so it’s best to avoid versions that are unreleased or that haven’t aged at least a few weeks.

### Some Gems Will Be Way More Difficult to Update

Large libraries, like Rails, RSpec, Twitter Bootstrap, etc. are going to take more elbow grease to update. Typically if a major version number is updating, like Rails 3.x to 4.x and RSpec 2.x to 3.x, that’s going to require lots of code changes. Semantic versioning also comes into play. Going from Rails 3.x to Rails 4.x is more difficult than Rails 4.0 to Rails 4.1\. There’s a similar story with RSpec 2.x to 2.99, compared to going to RSpec 3.0.


## Techniques for Smoother Gem Upgrades


### Locking Gem Versions

Unless you have a good reason, don’t lock a gem to a specific version as that makes updating gems more difficult. In general, consider only locking the major Rails gems, such as rails, RSpec, and bootstrap-sass, as these are the ones that will likely have more involved upgrades.

### Don’t Upgrade Major Libraries Too Soon


3 Reasons to wait a bit before gem updates:

1. **Dependencies among gem libraries** are not yet resolved. I had tried upgrading to RSpec 3 and Rails 4.1 a couple months ago, but it was apparent that I had to fix to many other gems to get them to work with RSpec 3\. Thus, I retreated back to RSpec 2.99 for a while. Now, as of August, 2014, the gem ecosystem was ripe to move to RSpec 3.0\. So unless you have a good reason, it’s best to wait maybe a couple of months after major upgrades are released before migrating.
2. **Bugs** may be lurking in changed code. If you wait a bit, the early adopters will find the bugs, saving you time and frustration. The more popular a gem, the faster it will be put to rigorous use.
3. **Security**/ problems may have been introduced. This is pretty much a special case of bugs, except that this a possibility of a malicious security change. If you wait a bit, hopefully somebody else will discover the issue first.

### Don’t Use Guard, Zeus, Spring, Spork, Etc. When Upgrading

Tools that speed up Rails like Zeus and Spring are awesome productivity enhancers, **except** when upgrading gems. I found that they _sometimes_ correctly reloaded new versions of gems. That means massive frustration when they are not picking up the gems you actually have specified. The corollary to this is to run your tests using plain `rspec` rather than the recommended ways for speeding up testing, such as the `parallel_tests` gem..

It’s not necessary to introduce the added complexity of the test accelerators when doing major library updates. Once you’ve updated your gems, then try out your favorite techniques for speeding up running tests. I’ve learned the hard way on this one. The `pgr` and `pgk` scripts below are awesome for ensuring that pre-loaders are _**NOT**_ running.


``` ruby
pgr() {
  for x in spring rails phantomjs zeus; do
    pgrep -fl $x;
  done
}

pgk() {
  for x in spring rails phantomjs zeus; do
    pkill -fl $x;
  done
}
```

### Tests: Try to Keep and Immediately Get Tests Passing

There are a lot of discussions about the value or lack of for an emphasis on Test-Driven Development (TDD). However, one thing that’s indisputable is that _**having a large library of tests is absolutely helpful for upgrading your gems**_.

Naturally, it’s an iterative process to get tests passing when updating gems. First, make sure your tests suite is passing.

You can try updating the gems one by one until you get a test failure. Then the issue becomes one of figuring out which related gems you might want to update to fix the test failure.

If you don’t have good tests coverage, a great place to start is with integration tests that do the basics of your app. At least you’ll be able to quickly verify a good chunk of your app can at least navigate the “happy path” as you iterate updating your gems.

### Alternate Big or Baby Steps

If you’ve updated gems recently, sometimes you can run `bundle update` and everything works great. Recently, that strategy failed miserably when I tried going from Rails 4.0 with RSpec 2.2 to Rails 4.1 and RSpec 3\. An eariler attempt shortly after the releases of Rails 4.1 and RSpec 3 clearly showed that many dependent gems would have to get updated. A few months later, I still had many issues with trying to update too much at once.

When this happens, take small steps and kept tests passing. I.e., don’t do a `bundle update` without specifying which gems to update. You might update 60 gems at once! And then when tests fail, you won’t be able to easily decipher which dependency is the problem. Specify which gems to update by running the command:


``` bash
$ bundle update gem1 gem2 etc
```

Then after updating a few gems, run `rspec` and verify your tests pass.

**Then commit your changes.** Consider putting a summary of how many tests pass and how long it takes. The length of time is useful in case some change greatly increases test run time. Or if you notice run time or the number of tests dramatically decrease. Plus, this ensures you ran the test before committing!

On a related note, you can see which gems are outdated with this command: `bundle outdated`.

### Try bundle update

Remember I told you not to do a `bundle update`? Once you’re getting closer to finishing your gem updates, all big gems are updated, and all tests are passing, and deprecation warnings are addressed, then it’s time to run `bundle update` and then run `rspec` to see if your tests pass. If you don’t have adequate tests, then be ready to do some adequate manual testing. Even if you have lots of tests, you still need to do manual testing if you upgrade a UI library such as `sass-bootstrap`. Besides testings, check the bundler output or the diff of your `Gemfile.lock` to see what got updated.

## Troubleshooting Gem Upgrades

### Read Error Messages Carefully and then search Google and Github Issues

Too often Ruby developers will blindly copy-paste their error messages into a Google search without really reading the console output carefully. This can actually waste more time, since thinking about the problem for a moment can often give you a solution without Google, or you’ll write a better search query. If you don’t find what you need on Google and you have an idea what gem is causing issues, the next place to search is the issues page for the gem’s Github repository.

Remember to do these 2 types of searches rather than spending too much time inserting print statements or launching the debugger! If you don’t get any search hits, then typically you have some problem in app customizations (see below).


### Visit the Gem Repository on Github

Some essential places to look at when upgrading gems are:

1. README.md file (shown on the main page of repository). Some projects might have a NEWS.md or CHANGELOG.md file.
2. The Github issues list for a gem (and search here)
3. The Github commit history for a gem, sometimes switching branches.

Errors or deprecation messages can come from compatibility issues among your gems. The RSpec 3 upgrade had many such issues. If you’re having an upgrade issue, then a concise, detailed post of a new issue typically results in a very quick response.


### Try an RC Version on RubyGems

Sometimes the fix you need has already been released to RubyGems in an RC version (RC means Release Candidate). `bundle update <gem>` seems to not pick RC versions. You have to specify these manually. I search for gems on RubyGems so often that I created a Chrome search shortcut. Here’s an example of an RC version gem that I’m currently using:


``` ruby
gem 'simple_form', '>= 3.1.0.rc2'
```


### Try a Github Gem Version Rather Than a RubyGems Version

Sometimes what you need has not been shared with RubyGems, yet the issue has received commits on Github. In that case, you can use the Github version of a gem. This might be on a specific branch of a gem, or even another user’s fork of a gem.

Here’s an example of the syntax to specify the very-latest version of a gem (the tip of the master branch):

``` ruby
gem 'gon', github: "gazay/gon", branch: "master"
```


Sometimes what you need is something less than the most current version, or a specific branch, or a fork of the gem.

### Consider Forking a Gem

Sometimes you need to fork a gem for some changes. If you’ve never done this, it’s a **very worthwhile thing to try out**, and it’s easy! For example, if you had wanted to update to rspec 3 sooner than later and didn’t want to see tons of deprecation messages, then your only option was to fork the gems that had the deprecated syntax. Once you’ve verified the validity of your changes, consider submitting a pull request. Here’s an example of a [fork and commit of the zeus-parallel_tests gem that loosened a gem dependency](https://github.com/justin808/zeus-parallel_tests/commit/ccd7367d4f33ae8940a4205a164df714ccfcb42c).

You should typically prefer a rubygems version of a gem rather than a github version. Thus, after some months, you should try to remove any previously necessary github references in your Gemfile.


### Order of Gems in your Gemfile Can Matter

I ran into a case where including rspec-instafail before rspec resulted in zeus failing due to `rspec-instafail` failing to recognize that I was using rspec 3\. Simply placing `rspec-instafail` after loading `rspec` in the Gemfile fixed that issue.

I had a clue that was the issue due to this stack dump. Note how the bundler is loading rspec-instafail, and when I looked at the source code, I could see why file `rspec_2.rb` was being loaded (2nd line of the below stack dump)


``` bash
zeus test
/Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/rspec-core-3.0.3/lib/rspec/core/formatters/progress_formatter.rb:1:in '<top (required)>': uninitialized constant RSpec::Support (NameError)
 from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/rspec-instafail-0.2.5/lib/rspec/instafail/rspec_2.rb:1:in '<top (required)>'
 from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/rspec-instafail-0.2.5/lib/rspec/instafail.rb:11:in '<module:RSpec>'
 from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/rspec-instafail-0.2.5/lib/rspec/instafail.rb:1:in '<top (required)>'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler/runtime.rb:85:in 'require'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler/runtime.rb:85:in 'rescue in block in require'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler/runtime.rb:68:in 'block in require'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler/runtime.rb:61:in 'each'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler/runtime.rb:61:in 'require'
 from /Users/justin/.rvm/gems/ruby-2.1.2@global/gems/bundler-1.6.2/lib/bundler.rb:132:in 'require'
 from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/zeus-0.13.3/lib/zeus/rails.rb:162:in 'test_environment'
 from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/zeus-0.13.3/lib/zeus.rb:166:in 'run_action'
  ...
 from -e:1:in '<main>'

```

### Evaluate Customizations

In general, when doing relatively major gem upgrades, you really need to evaluate customizations to these places. Typically, deprecation messages will tell you which customizations to remove or alter. Sometimes, you’ve monkey patched some gem to work around some issue, and this would be the place where you’d do that (and forget that you did it!).

1. Any initializers in the `config/initializers` directory. Review each file there.
2. Any customizations in your environment files in the `config/environments` directory, such as `test.rb`, `development.rb`.
3. Any customizations for running specs: a. `spec/spec_helper.rb` b. Each file in the `spec/support` directory.

## Example of Next Steps when Upgrading a Gem

Here’s an example of where updating related gems help.

`bundle update capybara` fixed the following error

``` bash
--------------------------------------------------------------------------------
Capybara::RSpecMatchers::HaveText implements a legacy RSpec matcher
protocol. For the current protocol you should expose the failure messages
via the `failure_message` and `failure_message_when_negated` methods.
--------------------------------------------------------------------------------
```

The final error I got was this one, from `cancan`.

``` bash
Deprecation Warnings:

'failure_message_for_should_not' is deprecated. Use 'failure_message_when_negated' instead. Called from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/cancan-1.6.10/lib/cancan/matchers.rb:11:in 'block in <top (required)>'.

'failure_message_for_should' is deprecated. Use 'failure_message' instead. Called from /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/cancan-1.6.10/lib/cancan/matchers.rb:7:in 'block in <top (required)>'.
```


A quick google search reveals that `cancancan` fixes the issue.

Once I got all tests passing, I tried to update to Rails 4.1, but ran into this issue:

``` bash
bundle update rails Fetching source index from https://rubygems.org/
Resolving dependencies........................
Bundler could not find compatible versions \for gem "activemodel":
  In Gemfile:
    simple_form (>= 0) ruby depends on
      activemodel (< 4.1, >= 4.0.0) ruby

    rails (> 4.1) ruby depends on
      activemodel (4.1.0)
```


I verify I’m on the current maximum GA version of simple_form, but I find that there’s an RC version, so I specify that in the gemfile. It’s important to note that “bundle update” will tend not to pull in RC versions of gems, which you sometimes need after major libraries are upgraded.

In `Gemfile`

``` ruby
gem 'rails', '~> 4.1'
gem 'simple_form', '>= 3.1.0.rc2'
```

``` bash
$ bundle update rails simple_form
Using rails 4.1.4 (was 4.0.8)
Installing simple_form 3.1.0.rc2 (was 3.0.1)
Your bundle is updated!
```

After the 4.1 upgrade, I addressed a number of deprecation warnings.

``` bash
DEPRECATION WARNING: Implicit join references were removed with Rails 4.1.Make sure to remove this configuration because it does nothing. (called from block in tsort_each at /Users/justin/.rvm/rubies/ruby-2.1.2/lib/ruby/2.1.0/tsort.rb:226)
```


``` ruby
config.active_record.disable_implicit_join_references = true
```

Then I got this warning with a full stack dump.

``` bash
Warning: you should require 'minitest/autorun' instead.
Warning: or add 'gem "minitest"' before 'require "minitest/autorun"'
From:
  /Users/justin/.rvm/gems/ruby-2.1.2@bpos/gems/activesupport-4.1.4/lib/active_support/dependencies.rb:247:in 'require'
```


The stack dump was useless, but the search for error message on Google found [this](https://github.com/thoughtbot/shoulda-matchers/issues/408) indicating that the issue had something to do with `shoulda-matchers`. A check of my gem version revealed that my gem version was not current.

``` bash
$ bundle update shoulda-matchers
Installing shoulda-matchers 2.6.2 (was 2.5.0)
```

And that fixed that issue!

I hope this article was useful for you. If you need help on your Rails projects, feel free to [connect](http://hackhands.com/justin808) to me on HackHands!

Please let me know if this article helped you or if I missed anything!

