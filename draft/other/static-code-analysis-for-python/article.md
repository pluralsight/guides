I have recently been tinkering around with
<a href="http://en.wikipedia.org/wiki/Static_code_analysis">static code analysis</a>
tools for python. The two I have tried out thus far are:

- [pychecker](http://pychecker.sourceforge.net/)
- [pylint](http://www.logilab.org/857)

## Pychecker

Pychecker is nice and simple and seems to work fairly well for standalone
projects such as my
<a href="https://github.com/durden/saucer_api">saucer_api</a> project.
However, it's simplicity was somewhat of a drawback for me because:

- It didn't play too well with <a
  href="http://www.djangoproject.com/">django</a> out of the box due to import
  problems. This is a fairly well <a
  href="http://www.google.com/search?q=pychecker+import+django">documented
  problem</a>.

- I really wanted something to check that I was adhering to some simple coding style standards such as variable/function naming consistency, line lengths, etc.

## Pylint

Pylint is very similar to to pychecker, but really steps it up a notch.
First, pylint doesn't have too many problems with
<a href="http://www.djangoproject.com/">django</a>. I did have to make some
modifications to get rid of errors regarding the use of generated django
attributes such as "objects" when calling "objects.all()", etc. on a model.
Luckily, this is easily done by:

1. Generating rcfile via pylint --generate-rcfile option
2. Copying default file to ~/.pylintrc
3. Modifying rcfile to include "objects" to "generated-members"

Second, pylint performs a much more detailed analysis of the code.  It verifies
code follows line length, naming consistency, etc.  It even gives your code a
score based on the errors, warnings, convention inconsistencies, etc.  This
makes it easy to say code is ready to be tested and/or checked in to the main
repository.  As previously mentioned, almost all these conventions, etc. can be
configured by an rcfile. This is convenient because the rcfile can also be
committed and serve as your standards documentation.

### Conclusion

<strong>Disclaimer:</strong><br/> I haven't used either of these tools very
extensively so I reserve the right to change my mind.  Also, I haven't
used either tool within python code yet.  My use has been restricted to
the command-line, which seems a more pragmatic way to use these
tools, because I simply want a quick, easy way to prepossess my code prior
to committing.

Both of these projects are open-source nice tools for the job.  However,
pylint was much better suited to my needs.  It is a bit overwhelming with
all of its output and metrics at first, but the majority of it is incredibly
useful.

- <strong>Use pychecker for easy way to find for syntax errors, etc.</strong>
- <strong>Use pylint for detailed syntax and coding standards verification</strong>

### Extra

Pychecker and pylint are not the only tools for python code analysis. I didn't
try
<a href="http://www.divmod.org/trac/wiki/DivmodPyflakes">pyflakes</a>, but
it seemed on the surface to be very similar to pychecker.

Stackoverflow has a nice <a href="http://stackoverflow.com/questions/1428872/pylint-pychecker-or-pyflakes">discussion</a> 
of these tools.
