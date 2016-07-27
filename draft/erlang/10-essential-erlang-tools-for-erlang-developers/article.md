## Introduction
What you'll find below are the tools that I use the most in my day-to-day life as an Erlang developer. The items in the list share the following characteristics:
- they're built specifically for Erlang (i.e. no text-editors, no git, etc.).
- they're not tailored to a particular kind of application (i.e. you can use them to develop any kind of Erlang application).

---
## Working in the Shell
Like many other languages, the most common way to interact with an Erlang VM is the Erlang shell. You can use it to test new things, to learn how to code and even to debug production systems. The Erlang shell is a powerful thing, but you can add much more power to it with the next 3 tools in this list:

![I see a blondie](http://img3.wikia.nocookie.net/__cb6/matrix/it/images/5/50/Wiki-background)

### 1. [user_default](http://erlang.org/doc/man/erl.html#id191234)
In general, to evaluate functions outside a module in Erlang you have to prepend their names with the corresponding module name. For example, if you want to sum up a list, you have to do the following:
```erlang
1> lists:sum([1, 2, 3]).
6
2>
```
But you'll notice that for certain functions in the shell, you don't need to do that, e.g.
```erlang
1> c(your_module).
{ok,your_module}
2> h().
1: c(your_module)
-> {ok,your_module}
ok
3> e(1).
{ok,your_module}
4>
```
Where do those functions live? They're defined in the [shell_default](https://github.com/erlang/otp/blob/maint/lib/stdlib/src/shell_default.erl) module. And you can add your own ones, too. To do that you have to create and load a module called user_default. As an example, one thing I like to do while in the Erlang shell is to run bash commands. You can do that using `os:cmd/1`, but that will give you the out as an Erlang string that you will need to print later to actually get a proper look at it. So, I added the following lines to my **user_default.erl** file:

```erlang
-module(user_default).
-export([cmd/1].

cmd(Cmd) -> io:format("~s~n", [os:cmd(Cmd)]).
```

Now I can do this in my shell:

```erlang
1> c(user_default).
{ok, user_default}
2> cmd("cat user_default.erl").
-module(user_default).
-export([cmd/1].

cmd(Cmd) -> io:format("~s~n", [os:cmd(Cmd)]).

ok
3>
```

### 2. [~/.erlang](http://erlang.org/doc/man/erl.html#id191234)
But then, I have to manually load `user_default` in every single shell that I open. Unless, I use **~/.erlang** for that! **~/.erlang** is a file that works almost like **~/.bashrc** for bash: When the shell starts, it reads **~/.erlang** and evaluates each one of the expressions it finds there as if they were input to the shell. So, I added the following lines to that file:
```erlang
UD = "/path/to/my/user_default".
io:format("~p~n", [shell_default:c(UD)]).
```
My **~/.erlang** is actually more complicated than that, but you get the picture, right?

### 3. [erlang-history](https://github.com/ferd/erlang-history)
One of the most annoying things about the Erlang shell is that, as provided by Erlang/OTP, it looses track of the history of commands you run on it as soon as you close it. To _fix_ that, in every single machine that I use for Erlang development, I install [erlang-history](https://github.com/ferd/erlang-history). It's a tiny almost-invisible hack to the Erlang/OTP distribution with one really simple purpose: to allow you to go back to your previous commands in your Erlang shell. Simple, yet amazingly useful.

---
## Managing your Apps
As soon as you move past the simple examples and into the realm of OTP apps, you'll be much better off working with a build tool that can help you organize your code, manage dependencies, build releases, run tests, etc.

![Dangerous to go Alone](http://www.tierragamer.com/wp-content/uploads/2016/02/its-dangerous-to-go-alone-take-this.jpg)

When working with Erlang, you can either embrace the _Makefile_ or try get as far away from it as possible.

### 4. [erlang.mk](https://erlang.mk/)
If using Makefiles to build and manage your projects seems natural to you, you will probably like [erlang.mk](https://erlang.mk/). **erlang.mk** is basically a big Makefile script that you can include in your own Makefile with something like:
```Makefile
PROJECT=your_app
include erlang.mk
```
With that, you get access to many commands, like
* `$ make` to build your project
* `$ make tests` to run your test suites
* `$ make rel` to generate a release

You can find the whole list using `$ make help`, and you can, of course, add more by yourself just extending your Makefile.
**erlang.mk** as well provides a plugin infrastructure, where you can add build tools, like [hexer.mk](https://github.com/inaka/hexer.mk) or [elvis.mk](https://github.com/inaka/elvis.mk).

The best thing about **erlang.mk** is that it provides support to build almost any existing Erlang library, regardless of the tool that library owners use to maintain it. With **erlang.mk** you can include in your deps applications built with **rebar**, **rebar3**, **erlang.mk** (of course), ad-hoc **Makefiles** and others. You can download deps from **github**, **hex.pm**, **bitbucket**, your local filesystem and even other places. **erlang.mk** is a very flexible tool.

### 5. [rebar3](http://www.rebar3.org/)
Now, if Makefiles are not your thing and/or you prefer to use _the official build tool_ (since March 2016) sponsored by the Erlang/OTP team, then [rebar3](http://www.rebar3.org/) is your tool.
**rebar3** is written entirely in Erlang and the idea is that you can use it _without_ adding any Makefile to your project.
Once you [install it](http://www.rebar3.org/v3.0/docs#installing-binary) in your system, you should add a `rebar.config` file in the root folder of your project. A minimal one looks something like this (although even the options below are optional):
```erlang
{deps, []}.
{erl_opts, [debug_info]}.
{cover_enabled, true}.
```
You can find the list of all the options you can speficy [here](http://www.rebar3.org/docs/configuration).
Then, you can run commands like the ones below in your shell:
* `$ rebar3 compile` to fetch deps and build your project
* `$ rebar3 ct` to run your tests
* `$ rebar3 release` to generate a release

Of course, `$ rebar3 help` gives you the list of all available commands. And, just like **erlang.mk**, **rebar3** is extensible through [plugins](http://www.rebar3.org/docs/using-available-plugins).

One of the best things about **rebar3** is that, by using rebar.lock, it provides _repeatable builds_. That way, once you're sure your project works as expected, you can be sure it will keep working as expected no matter how many times or in how many different places you build it.

---
## Debugging your Servers
### 6. redbug

### 7. recon

---
## Ensuring Code Quality
### 8. dialyzer

### 9. xref

### 10. elvis

---
## Bonus Tracks
### 11. Meta Testing

### 12. The Community