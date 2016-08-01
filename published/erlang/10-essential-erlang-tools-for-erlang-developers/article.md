## Introduction
In this tutorial, I'll cover the tools that I use the most in my day-to-day life as an Erlang developer. All or most of the topics that make the cut share the following characteristics:
- they're built specifically for Erlang (i.e. no text-editors, no git, etc.).
- they're not tailored to a particular kind of application (i.e. you can use them to develop _any kind_ of Erlang application).

---
## Working in the Shell
Like many other languages, the most common way to interact with an Erlang VM is through the Erlang shell. You can use it to test code, learn how to use Erlang, and debug production systems. The Erlang shell is a powerful thing, but you can add much more power to it with the three tools that I will show you below.

![I see a blondie](https://c2.staticflickr.com/8/7059/6886658738_d371a49e54_b.jpg)

### 1. [user_default](http://erlang.org/doc/man/erl.html#id191234)
In general, to evaluate functions outside a module in Erlang you have to prepend their names with the corresponding module name. For example, if you want to sum up a list, you have to do the following:
```erlang
1> lists:sum([1, 2, 3]).
6
2>
```
But you'll notice that for certain functions in the shell, you don't need explicit definitions. For example:
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
Where do such functions live? They're defined in the [shell_default](https://github.com/erlang/otp/blob/maint/lib/stdlib/src/shell_default.erl) module. You can add your own default functions as well. To do that, you have to create and load a module called `user_default`. As an example, I like to run bash commands when using the Erlang shell; I don't want to leave the shell just to copy a file. Erlang/OTP gives you a a function to do that: `os:cmd/1`, but that will not print out the output of your command. Instead this command will return that output as a string. You have to print the string to see the output properly.

So, using **`user_default`**, I added the following lines to my **`user_default.erl`** file:

```erlang
-module(user_default).
-export([cmd/1].

cmd(Cmd) -> io:format("~s~n", [os:cmd(Cmd)]).
```

Now I can run, execute, and *print* the results of `os:cmd/1` using my shell:

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
But then, I have to manually load `user_default` in every single shell that I open. **`~/.erlang`** to the rescue! `~/.erlang` is a file that works almost like `~/.bashrc` for bash. Upon starting, the shell reads `~/.erlang` and evaluates each one of the expressions it finds there as if they were inputted to the shell. 

So, I added the following lines to that file:
```erlang
UD = "/path/to/my/user_default".
shell_default:c(UD).
```
My `~/.erlang` is actually more complicated than that, but the `shell_default` should give you the gist of my modifications. Now, every shell that I open comes with support for `cmd/1`.

### 3. [erlang-history](https://github.com/ferd/erlang-history)
One of the most annoying things about the Erlang shell is that it does not keep track of command history very well. In fact, default settings will erase all command history upon closing the shell. I've seen many devs working around this limitation by having a notebook of erlang expressions from which they copy & paste into the console every time. (Some of them use `user_default` and/or `~/.erlang` for that.) 

But still, it's not a real solution, right? To properly fix the issue, right after installing a new version of Erlang/OTP, **I install [erlang-history](https://github.com/ferd/erlang-history).** It's a really tiny almost-invisible hack to the Erlang/OTP distribution with one simple purpose: to keep track of your previous commands in your Erlang shells and let you reuse them. Simple, yet amazingly useful.

---
## Managing your Apps
As soon as you move past the simple examples and into the realm of OTP apps, you'll be much better off working with a build tool that can help you organize your code, manage dependencies, build releases, run tests, and more.

![Dangerous to go Alone](http://www.tierragamer.com/wp-content/uploads/2016/02/its-dangerous-to-go-alone-take-this.jpg)

When working with Erlang, you can either embrace the _Makefile_ or try get as far away from it as possible.

### 4. [erlang.mk](https://erlang.mk/)
If using Makefiles to build and manage your projects seems natural to you, you will probably like [erlang.mk](https://erlang.mk/). `erlang.mk` is basically a big Makefile script that you can include in your own Makefile with something like:
```Makefile
PROJECT=your_app
include erlang.mk
```
With that, you get access to many commands, like
* `$ make` to build your project
* `$ make tests` to run your test suites
* `$ make rel` to generate a release

You can find the whole list using `$ make help`, and you can, of course, add more by just extending your Makefile.

`erlang.mk` also provides a plugin infrastructure, where you can add build tools, like [hexer.mk](https://github.com/inaka/hexer.mk) or [elvis.mk](https://github.com/inaka/elvis.mk).

The best thing about `erlang.mk` is that it provides the necessary support for building almost any existing Erlang library, regardless of the tool that library owners use to maintain it. With `erlang.mk` you can include applications built with **rebar**, **rebar3**, **erlang.mk** (as expected :P), ad-hoc **Makefiles**, and others in your depositories. You can download deps from **github**, **hex.pm**, **bitbucket**, your local filesystem and many other places. Clearly `erlang.mk` makes Erlang vastly more versatile.

### 5. [rebar3](http://www.rebar3.org/)
Now, if Makefiles are not your thing and you prefer to use _the official build tool_ (since March 2016) sponsored by the Erlang/OTP team, then [rebar3](http://www.rebar3.org/) is your way to go.

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

Of course, `$ rebar3 help` gives you the list of all available commands. And, just like `erlang.mk`, `rebar3` is also extensible through [plugins](http://www.rebar3.org/docs/using-available-plugins).

One of the best things about `rebar3` is that, by using `rebar.lock`, it provides _repeatable builds_. That way, once you're sure your project works as expected, you can be sure it will keep working as expected no matter how many times or in how many different places you compile it.

---
## Ensuring Code Quality
I've personally stated a couple of times online (in [interviews](https://www.functionalgeekery.com/functional-geekery-episode-43-brujo-benavides/) and blog posts) how important it is (especially if you work on open-source projects) to maintain high code quality. Erlang comes with several tools that will help you with that. Erlang tools can even help you detect bugs that you'll probably find at runtime… without running your system!

![Paddin](https://i.ytimg.com/vi/8ktYyme_sUw/maxresdefault.jpg)

### 6. [dialyzer](http://erlang.org/doc/man/dialyzer.html)
**Dialyzer** is a static analysis tool that will help you identify type inconsistencies in your code, like when you are passing a `binary()` as the parameter to a function that expects a `string()`. This used to be a very very slow tool, but it has since been highly optimized and now runs smoothly, especially if you run it frequently. Plus, it's integrated in both [`rebar3`](https://www.rebar3.org/docs/commands#dialyzer) and [`erlang.mk`](https://erlang.mk/guide/dialyzer.html). (Follow the links to learn more.)

If you use **dialyzer** (`dia.erl`), you'll see results like the one below.
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
In this case in line 5 of `dia.erl`, we are calling `lists:flatten/1` with a 168-bit long binary (`<<_:168>>`) when that function expects a list (`[any()]`) as its parameter. This warning was **not** raised at compile time, but using **dialyzer** you can find it **before** it pops up during run time.

### 7. [xref](http://erlang.org/doc/man/xref.html)
A much simpler tool that's been packed in the Erlang/OTP distribution is **`xref`**. `xref` is a cross-reference tool that analyses your code and finds calls to unexistent functions, deprecated functions and other similar things. It doesn't search as deeply as dialyzer, but it will spot obvious bugs in no time. 

`xref` is also integrated with [`rebar3`](https://www.rebar3.org/docs/commands#xref) and [`erlang.mk`](https://erlang.mk/guide/xref.html) (The documentation is not there yet, but it uses [`xref_runner`](https://github.com/inaka/xref_runner) to check your code).


`xref` can find errors like:
```erlang
…
===> Running cross reference analysis...
===> Warning: your_module:your_function/1 calls undefined function this_function:doesnt_exist/1 (Xref)
…
```
There `xref` detected that we're calling a function that doesn't exist from within the code of `your_module:your_function/1`. Again, that's something that the compiler __will not__ warn you about.

### 8. [elvis](http://github.com/inaka/elvis)
Besides analyzing your code in search for bugs with `xref` and dialyzer, you can ensure that the code in your project is maintainable. **elvis** is a command-line tool (and an Erlang application as well) that you can use to verify the compliance of your code to certain style rules, which you pre-define in your `elvis.config` file. There is a plugin for [`rebar3`](https://github.com/project-fifo/rebar3_lint) and another one for [`erlang.mk`](https://github.com/inaka/elvis.mk). You can also use **elvis [online](http://elvis.inakalabs.com)** to check your github pull requests.


Its output looks like the following:
```erlang
Loading files...
Loading src/dia.erl
Applying rules...
# src/dia.erl [FAIL]
  - no_tabs
    - Line 33 has a tab at column 0.
```
In this case, elvis is detecting the usage of tabs for indentation which is something that's specified as invalid in the project's `elvis.config` file.

---
## Debugging your Servers
The next group of tools assume that you already created your app, packed it up as a release, and deployed it to some server. They also assume that the app's been running for a while, but has started behaving strangely. So you need to _debug_ it. 

First, you connect to your server with a [remote shell](http://erlang.org/doc/man/erl.html#id186912) (See the link for the `-remsh` command). Then, you use Erlang's [dbg](http://erlang.org/doc/man/dbg.html). 

Alas, **dbg** is not a very intuitive, simple debugging tool. It's really powerful, but there are simpler tools for the task at hand. I'll show you 2 of them below.

![investigating…](https://upload.wikimedia.org/wikipedia/commons/1/13/Special_Agent_Adam_Deem_of_Air_Force_Office_of_Special_Investigation_Detachment_219.jpg)

### 9. [redbug](https://hex.pm/packages/eper)
**redbug** is a debugging application which is quite similar to **dbg**, but with a more intuitive interface. It comes with **eper** but, to be honest, `redbug` is the only `eper` component I've ever used. With `redbug`, you can specify a trace pattern as a string, and it will print out a message in your console everytime a function call matching that pattern is evaluated. It looks like this:

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

I actually tend to always use the same options when starting `redbug`. You can check the available startup options using `redbug:help()`. As such, I've added this start code to my [user_default](#1-a-href-http-erlang-org-doc-man-erl-html-id191234-target-_blank-user_default-nbsp-span-class-glyphicon-glyphicon-new-window-aria-hidden-true-style-font-size-10px-span-a-) (Don't do this in production, kids!!):

```
redbug(What) ->
  catch redbug:stop(),
  timer:sleep(100),
  redbug:start(What, [{time, 9999999}, {msgs, 9999999}, {print_msec, true}]).
```

Now I can easily get `redbug` going.:

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
But tracing function calls is just one of the many things you can do to check the health of your Erlang servers. [Fred Hebert](https://twitter.com/mononcqc/) recollected tons of experiences dealing with that in both a great book ([Stuff Goes Bad: Erlang In Anger](https://www.erlang-in-anger.com/)) and a great tool: [**recon**](http://ferd.github.io/recon/).


With **recon** you can do many things, you can: trace function calls (just like `redbug`), recover the source code of your compiled modules, check app memory consumption, figure out message queue lengths, clean up binary memory, find leaks and much more. I strongly recommend you to read the book and play around with the different `recon` tools described in it. You'll want to know them by heart if your server starts behaving erratically.

---
## Bonus Tracks
Now some pointers that don't _really_ fit in the list above, but deserve to be mentioned anyway.

![Bonus](http://www.picserver.org/images/highway/phrases/bonus.jpg)

### 11. [Meta Testing](https://github.com/inaka/katana-test)
If you use TDD (even if you just use [common test](http://erlang.org/doc/man/common_test.html) to test your Erlang apps and you are interested in maintaining your [code quality](#ensuring-code-quality) using the tools I mentioned above, you have to check [katana-test](https://github.com/inaka/katana-test). With the code below, you'll be dialyzing, `xref`ing, and rocking your code every time you run your tests.

```erlang
-module(your_meta_SUITE).

-include_lib("mixer/include/mixer.hrl").
-mixin([ktn_meta_SUITE]).

-export([init_per_suite/1]).

init_per_suite(Config) -> [{application, your_app} | Config].
```

I call that **Meta Testing**, and I've written two blog posts about it already. Check them out [here](http://inaka.net/blog/2015/07/17/erlang-meta-test/) and [here](http://inaka.net/blog/2015/11/13/erlang-meta-test-revisited/).

### 12. The Community
No list of great Erlang things will be complete without a shout out to the great community of developers behind it. If you're going to work on Erlang or even if you're just approaching the language and you want to know what it's all about, _you have to meet the erlangers_. 

There are a couple of ways to acoomplish that:
* The [erlang-questions](http://erlang.org/mailman/listinfo/erlang-questions) mailing list is the most popular and arguably the _official_ communication medium for the community. Yes, it's a mailing list. No, it will not fill up your inbox like spam.
* We also have an [IRC room](http://irc.lc/freenode/erlang/t4nk@@@) on _freenode_ and a [ server](erlanger.slack.com) on Slack where we constantly meet.
* And you can find some of us on [ErlangCentral](http://erlangcentral.org), too.

If you're serious about Erlang development, keep an eye on those platforms and stay connected with us for updates and other opportunities!

### 13. [Guidelines](https://github.com/inaka/erlang_guidelines)

Finally, I want to mention the Erlang Guidelines, published by Inaka [on github](https://github.com/inaka/erlang_guidelines). As I said [in the mailing list](http://erlang.org/pipermail/erlang-questions/2014-September/080935.html) when we first talked about them:
> [erlang_guidelines] is the set of standards that we (at Inaka) enforce in our Erlang code.
> […]
> It will only reflect the ideas of the ~8 erlangers that were/are working at Inaka. We accept contributions because we love good and constructive debate, but we don’t expect anybody […] to like [our guidelines] :)

In other words, the guidelines on that repo don't need to be **your** guidelines, but you should have **some** guidelines of your own. Forking that repo and then adapting its contents to your ideas may be a good way to start. Others have done [exactly that](https://github.com/inaka/erlang_guidelines/network) already.
As [Iñaki](https://twitter.com/iraunkortasuna) once explained:

> Code formatting, spacing, layout and _
"making the code look good"_ is **not** the goal of having guidelines. Similarly, the idea is not to engage in bondage programming in which everyone is forced to do something they don't want, but to have a baseline of order and agreed-upon-terms.
>
> The purpose of having consensus and following it is to **free up energy**, not to consume it in favor of poorly-defined or subjective goals such as _"it looks better"_. Having each one one make up his/her own rules _"and damn the rest"_ is just as antisocial as forcing everyone to adhere to strict guidelines which dictate where you put your commas or spacing. Discipline provides freedom
to the group, because once you achieve consensus on a matter, the effort of dealing with other's trespasses is greater than the effort of keeping oneself from trespassing.
>
> Issues such as line length restrictions are a means to an end: they make the code easier to change, debug, and extend to respond faster to requirement changes. To agree with any of this, one must acknowledge that programming is a **social endeavor**, and group behaviour applies. It may be easier to write for you, at this time, but others will have to read it. This is the sole objective of code guidelines.

___

Hopefully this guide cleared up some of the questions you may have had regarding Erlang-based application development and also boosted your interest in creating Erlang apps! Thank you for reading, and please leave your comments and feedback in the discussion section below. 