## Introduction
What you'll find below are the tools that I use the most in my day-to-day life as an Erlang developer. The items in the list share the following characteristics:
- they're built specifically for Erlang (i.e. no text-editors, no git, etc.).
- they're not tailored to a particular kind of application (i.e. you can use them to develop any kind of Erlang application).

---
## Working in the Shell
Like many other languages, the most common way to interact with an Erlang VM is the Erlang shell. You can use it to test new things, to learn how to code and even to debug production systems. The Erlang shell is a powerful thing, but you can add much more power to it with the next 3 tools in this list:

![I see a blondie](https://c2.staticflickr.com/8/7059/6886658738_d371a49e54_b.jpg)

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
## Ensuring Code Quality
I've personally stated a couple of times online (in interviews and blog posts) how important it is (especially if you work on open-source projects) to keep your code quality high. Erlang comes with several tools that will help you with that. They'll even help you detect bugs that you'll probably miss at compile time!

![Paddin](https://i.ytimg.com/vi/8ktYyme_sUw/maxresdefault.jpg)

### 6. [dialyzer](http://erlang.org/doc/man/dialyzer.html)
**Dialyzer** is a static analysis tool that will help you identify type inconsistencies in your code, like when you are passing a `binary()` as the parameter to a function that expects a `string()`. It used to be a very very slow tool but it has been greatly optimized througout the years and now it runs smoothly, specially if you run it frequently. And it's integrated in both [rebar3](https://www.rebar3.org/docs/commands#dialyzer) and [erlang.mk](https://erlang.mk/guide/dialyzer.html).
If you use **dialyzer**, you'll see results like the one below.
```erlang
  Checking whether the PLT dia.plt is up-to-date... yes
  Proceeding with analysis...
    compile    (+0.09s):   0.02s (   1 modules)
    clean      (+0.00s):   0.00s
    remote     (+0.00s):   0.18s
    order      (+0.00s):   0.00s
    typesig    (+0.12s):   0.01s (      3 SCCs)
    order      (+0.00s):   0.00s
    refine     (+0.00s):   0.01s (   1 modules)
    warning    (+0.00s):   0.00s (   1 modules)
              (+ 0.71s)

dia.erl:5: Function bad/0 has no local return
dia.erl:5: The call lists:flatten(<<_:168>>) will never return since it differs in the
            1st argument from the success typing arguments: ([any()])
 done in 0m1.16s
done (warnings were emitted)
```

### 7. [xref](http://erlang.org/doc/man/xref.html)
A much simpler tool that also comes packed in the Erlang/OTP distribution is **xref**. **xref** is a cross-reference tool that analyses your code and finds calls to unexistent functions, deprecated functions and other things. It doesn't go as deep as **dialyzer**, but it will spot obvious bugs in no time. It's also integrated with [rebar3](https://www.rebar3.org/docs/commands#xref) and [erlang.mk](https://erlang.mk/guide/xref.html) (The documentation is not yet there, but it uses [xref_runner](https://github.com/inaka/xref_runner)).
Using **xref** you'll find things like:
```erlang
…
===> Running cross reference analysis...
===> Warning: your_module:your_function/1 calls undefined function this_function:doesnt_exist/1 (Xref)
…
```

### 8. [elvis](http://github.com/inaka/elvis)
Besides analysing your code in search for bugs with **xref** and **dialyzer**, you can also use a tool to ensure that the code in your project respect your standards. For that, you have **elvis**. **elvis** is a command-line tool (and an Erlang application as well) that you can use to verify the compliance of your code to certain style rules you can define in your `elvis.config` file. There is a plugin for [rebar3](https://github.com/project-fifo/rebar3_lint) and another one for [erlang.mk](https://github.com/inaka/elvis.mk). You can also use **elvis** [online](http://elvis.inakalabs.com) to check your github pull requests.
**elvis** produces output like the following one:
```erlang
Loading files...
Loading src/dia.erl
Applying rules...
# src/dia.erl [FAIL]
  - no_tabs
    - Line 33 has a tab at column 0.
```

---
## Debugging your Servers
The next group of tools assume that you already created your app, packed it up as a release and deployed it to some server. It's been running for a while but now it's behaving in a strange manner. So, you need to _debug_ it. You connect to your server with a [remote shell](http://erlang.org/doc/man/erl.html#id186912) (look for -remsh there) and then you can use Erlang's [dbg](http://erlang.org/doc/man/dbg.html), but there are simpler tools for that job. I'll show you 2 of them below.

![investigating…](https://upload.wikimedia.org/wikipedia/commons/1/13/Special_Agent_Adam_Deem_of_Air_Force_Office_of_Special_Investigation_Detachment_219.jpg)

### 9. [redbug](https://hex.pm/packages/eper)
**redbug** is a debugging application which is quite similar to **dbg**, but with a nicer interface. It comes with **eper** but, to be honest, **redbug** is the only one of the **eper** apps I've ever used. With **redbug** you can specify a trace pattern as a string and it will print out a message in your console everytime a function call matching that pattern is evaluated. It looks like this:

```erlang
1> redbug:start("your_module:your_private_function->return").
{156,1}
2> your_module:your_public_function().
% 19:55:12 <0.1.0>({erlang,apply,4})
% your_module:your_private_function(some, parameters)

% 19:55:12 <0.1.0>({erlang,apply,4})
% your_module:your_private_function/2 -> ok
ok
3>
```

I actually tend to always use the same options when starting redbug, so I added this code to my [user_default](#1-a-href-http-erlang-org-doc-man-erl-html-id191234-target-_blank-user_default-nbsp-span-class-glyphicon-glyphicon-new-window-aria-hidden-true-style-font-size-10px-span-a-) (Don't do this in production, kids!!):

```
redbug(What) ->
  catch redbug:stop(),
  timer:sleep(100),
  redbug:start(What, [{time, 9999999}, {msgs, 9999999}, {print_msec, true}]).
```

Now I can easily do…

```erlang
4> redbug("your_module:your_private_function->return").
{156,1}
5> your_module:your_public_function().
% 19:59:13 <0.1.0>({erlang,apply,4})
% your_module:your_private_function(some, parameters)

% 19:59:13 <0.1.0>({erlang,apply,4})
% your_module:your_private_function/2 -> ok
ok
6>
```

### 10. [recon](https://hex.pm/packages/recon)
But tracing function calls is just one of the many things you can do to check the health of your Erlang servers. [Fred Hebert](https://twitter.com/mononcqc/) collected tons of experience dealing with Erlang servers in both a great book ([Stuff Goes Bad: Erlang In Anger](https://www.erlang-in-anger.com/)) and a great tool: [**recon**](http://ferd.github.io/recon/).
With **recon** you can do many things, you can trace function calls (a'lla **redbug**), you can recover the source code of your compiled modules, you check the memory consumption, message queue lengths and other important statistics of your nodes, you can clean up binary memory, etc. I recommend you to read the book and try the different **recon** tools described in it. You'll want to have them near if your server starts behaving erratically.

---
## Bonus Tracks
And now a couple of things/tools/tips that doesn't _really_ fit in the list above but deserve to be mentioned anyway.

![Bonus](http://www.picserver.org/images/highway/phrases/bonus.jpg)

### 11. [Meta Testing](https://github.com/inaka/katana-test)
If you use TDD (even if you just use [common test](http://erlang.org/doc/man/common_test.html) to test your Erlang apps) and you are interested in maintaining your [code quality](#ensuring-code-quality) using the tools I mentioned above, you have to check [katana-test](https://github.com/inaka/katana-test). With as little code as I will show you below, you'll be dialyzing, xref'ing and rocking your code every time you run your tests.

```erlang
-module(your_meta_SUITE).

-include_lib("mixer/include/mixer.hrl").
-mixin([ktn_meta_SUITE]).

-export([init_per_suite/1]).

init_per_suite(Config) -> [{application, your_app} | Config].
```

I call that **Meta Testing**, and I've wrote [two](http://inaka.net/blog/2015/07/17/erlang-meta-test/) [blog](http://inaka.net/blog/2015/11/13/erlang-meta-test-revisited/) posts about it already. Check them out.

### 12. The Community
No list of great Erlang things will be complete without a mention of the great community of developers behind it. If you're going to work on Erlang or even if you're just approaching the language and you want to know what it's all about, **you have to meet the erlangers**. There are a couple of ways to acoomplish that:
* The [erlang-questions](http://erlang.org/mailman/listinfo/erlang-questions) mailing list is the most popular and arguably the _official_ communication medium for the community. Yes, it's a mailing list. No, it will not fill up your inbox like spam.
* 

### 13. [The Guidelines](https://github.com/inaka/erlang_guidelines)

