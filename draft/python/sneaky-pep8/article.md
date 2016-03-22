I'm a pretty strong [PEP8](http://legacy.python.org/dev/peps/pep-0008/)
advocate. Yet, there are definitely times when I stray from the standard a
bit, most notably when breaking lines to fit within 79 characters.  However, I
appreciate there's a standard for [Python](http://python.org) and do my best to
live up to that.

Often times my team members and I fall short of this standard, and I find
myself *fixing* up the code to be compliant.  Possible regressions and
*meaningless* diffs in the project repository are the biggest downsides of this
practice.  For example, I might be hunting a bug with
[git bisect](https://www.kernel.org/pub/software/scm/git/docs/git-bisect.html)
and waste a bunch of time stepping through commits that are just style changes.
[1]

The ideal situation is that style *fixes* only occur when you're already
editing a specific section of code.  Thus, you don't have a bunch of commits
that are *purely* syntax tweaks.  This is essentially the
[Boy Scout Rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule) of Software Engineering.

Sounds like a good idea, right? Well [Andy Hayden](https://github.com/hayd)
probably agrees and so he wrote some code to do just that, fix PEP8 style
issues around already modified code.  Enter
[pep8radius](https://github.com/hayd/pep8radius).

It's a great idea, and pep8radius plays nicely with [git](http://git-scm.com/)
and several other VCS systems.  The script simply looks at the most recent diff
and attempts to cleanup style issues without introducing any extra noise/lines
to the diff.

Now you can slowly sneak PEP8 compliance into your project!

[1] To be fair, this isn't time completely wasted since style changes can
    easily inject their own bugs if you're not careful.
