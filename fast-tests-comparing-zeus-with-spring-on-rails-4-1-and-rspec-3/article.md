What’s faster? [Zeus](https://github.com/burke/zeus) with [Parallel Tests](https://github.com/grosser/parallel_tests) or [Spring](https://github.com/rails/spring), in the context of Rails 4.1, RSpec 3, Capybara 2.4, and PhantomJs

**The bottom line is that both work almost equivalently as fast, and the biggest difference for me concerned compatibility with the parallel_tests gem**. Zeus works fine with Parallel Tests, although it makes little difference overall with or without Zeus. Spring doesn’t work with Parallel Tests, but you can work around this issue. So stick with Zeus if it works for you. And regardless of using Spring or Zeus, the shell scripts provided below called `pgr` and `pgk` are essential for quickly listing or killing Zeus, Spring, Rails, or Phantomjs processes! It’s also worth noting that biggest advantage of using the Zeus or Spring pre-loaders is to save the Rails startup time. On my machine, this is about 3 to 5 seconds. That matters a lot if the test I’m focusing upon only takes a second or two, such as when doing TDD. However, when running a whole test suite taking minutes, 3-5 seconds can get swallowed up by other things, such as rspec-retry, which retries failing capybara tests.

## Overview

Both Spring and Zeus preload Rails so that you don’t have to pay the same start up cost for every test run. Here’s a RailsCast on the topic: [#412 Fast Rails Commands](http://railscasts.com/episodes/412-fast-rails-commands).

The end results is that both Zeus and Spring give great results and are very similar in many ways. The biggest difference for me is that only Zeus (and not Spring) works with Parallel Tests. Interestingly, I got very similar results when using Parallel Tests with our without Zeus. It turns out that it is possible to run Parallel Tests with Spring installed so long as you disable it by setting the environment variable like this:

``` bash
$ DISABLE_SPRING=TRUE parallel_rspec -n 6 spec
```
The bottom line for me is that I don’t have any good reason to move away from Zeus to Spring, and the fact that Spring is part of stock Rails is not a sufficient reason for me. That being said, on another project which is smaller, I’m not motivated to switch from Spring to Zeus.

## Performance

Note in below commands, one must insert `zeus` in the command to be using zeus. If using Spring, be sure that you’re using the Spring modifed binstub scripts in your bin directory by having your path appropriately set or using `bin/rake` and `bin/rspec` (install [spring-commands-rspec](https://github.com/jonleighton/spring-commands-rspec)). The times shown below are from both sample runs of a single directory of non-integration specs and from the full test suite of 914 tests, many of which are Capybara tests, on a 2012, Retina, SSD, 16 GB, MacBook Pro while running Emacs, RubyMine, Chrome, etc. Times were gathered by running commands prefixed with the `time` command. Running `zeus rspec` seems a bit slower than using spring. However, when running the integration tests, my test execution time was always variable depending on the number of Capybara timeouts and retries.

| command | zeus loader | spring loader | no loader |
|---------------------------------------------------|
| rspec spec/utils | 0:19.1 | 0:17.7 | 0:22.8 |
| rake spec:utils | 0:15.6 | 0:17.9 | 0:18.1 |
| rake spec | 6:11.9 | 6:15.0 | 8:02.5 |
| rspec spec | 5:51:7 | 5:28.0 | 5:37.2 |
| parallel_rspec -n 6 spec | 2:28.7 | n/a | 2:28.0 |


## Zeus and Spring vs. plain RSpec

Here’s some advantages and disadvantages of using either either Zeus or Spring compared to plain RSpec.

### Advantages

1. Both save time for running basic commands like rspec, rake, rails, etc. The performance of both is very similar.

### Disadvantages

1. Both can be extremely confusing when they fail to update automatically. This tends to happen after updating gems or running database migrations. You end up yak shaving when you don’t see your changes taking effect! I.e., put in some print statements, and then you don’t see them shown when they should. Arghhhh!
2. [Rspec-retry](https://github.com/y310/rspec-retry) seems essential in dealing with random Capybabara failures with either Zeus or Spring. I often see less of these errors when I don’t use Zeus/Spring nor parallel_tests.

## Zeus vs. Spring

### Advantages

1. [Zeus](https://github.com/burke/zeus) works with the [parallel_tests](https://github.com/grosser/parallel_tests) gem. This more than halves my time for running my entire test suite. However, when writing this article, I found that it made little difference, at least when slowed down by sporadically failing capybara tests that are retried. That being said, I’m certain that Parallel Tests with Zeus is faster or at worse the same as without Zeus.

### Disadvantages

1. You need to start up separate shell process, running `zeus start`. An advantage of this is that if there’s a problem starting up, the output in the Zeus console window is fairly clear.
2. You run the command “zeus rake” rather than just “rake”. Consequently, I made some shell aliases (see below).
3. Zeus only uses the environment from when Zeus was started and ignores any environment variables when commands are run.

## Spring vs. Zeus

### Advantages

1. [Spring](https://github.com/rails/spring) is a default part of Rails, so you know it’s well supported, and bugs will be fixed fast.
2. Slightly simpler to install and use than Zeus.

### Disadvantages

1. Spring lacks support for parallel_tests. See this Github issue: [incompatible with spring #309](https://github.com/grosser/parallel_tests/issues/309#issuecomment-45056130). You can, however run parallel_tests so long as run the command like this: `time DISABLE_SPRING=TRUE parallel_rspec -n 6 spec`. I.e., you need to set `DISABLE_SPRING` so that parallel_rspec does not use Spring.
2. Spring is a bit opaque in terms of errors given there’s no console window. See [README](https://github.com/rails/spring) for how to see the Spring log.

## Miscellaneous Tips

Be sure to disable either Zeus or Spring when updating gems. Consider restarting Zeus or Spring after a database migration. See the below scripts called `pgr` and `pgk` for seeing and killing Zeus/Spring related processes.

## Relevant Gems Working For Me

The right combination of gems seem pretty critical in getting all the parts to play nicely together. As of August 15, 2014 the most recent compatible versions of the following gems worked well together. This means running “bundle update” without locking the gem versions.

> capybara-screenshot (0.3.21) capybara (2.4.1) guard (2.6.1) guard-bundler (2.0.0) guard-livereload (2.3.0) guard-rails (0.5.3) guard-resque (0.0.5) guard-rspec (4.3.1) guard-unicorn (0.1.1) parallel_tests (1.0.0) poltergeist (1.5.1) rails (4.1.4) resque_spec
> (0.16.0) rspec (3.0.0) rspec-instafail (0.2.5) rspec-its (1.0.1) rspec-mocks (3.0.3) rspec-rails (3.0.2) rspec-retry (0.3.0) vcr (2.9.2) webmock (1.18.0) zeus (0.13.3) zeus-parallel_tests (0.2.4)


## Zeus Shell Configuration (ZSH)

``` bash
echoRun() {
  START=$(date +%s)
  echo "> $1"
  eval time $1
  END=$(date +%s)
  DIFF=$(( $END - $START ))
  echo "It took $DIFF seconds"
}

alias zr='zeus rake'

alias parallel_prepare='rake parallel:prepare ; "rake parallel:rake\[db:globals\]" '

zps() {
  # Run parallel_rspec, using zeus, passing in number of threads, default is 6

  p=${1:-6}
  # Skipping zeus b/c env vars don't work with zeus

  # start zeus log level fata
  # echoRun "SKIP_RSPEC_FOCUS=YES RSPEC_RETRY_COUNT=7 RAILS_LOGGER_LEVEL=4 zeus parallel_rspec -n $p spec"
  echoRun "zeus parallel_rspec -n $p spec"
}

# List processes related to rails
pgr() {
  for x in spring rails phantomjs zeus; do
    pgrep -fl $x;
  done
}

# Kill processes related to rails
pgk() {
  for x in spring rails phantomjs zeus; do
    pkill -fl $x;
  done
}
```


## Conclusion

Thanks for reading. I hope this article was useful for you! If you'd like custom help with Zeus, Spring, or Parallel Tests, I'm available for custom help [here](https://hackhands.com/justin808).

