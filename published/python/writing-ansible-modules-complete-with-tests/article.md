NOTE: This guide IS OUTDATED. Please visit [this site](https://www.relaxdiego.com/2016/06/writing-ansible-modules-with-tests.html) for a more updated version.

I'm writing an Ansible module that I hope to contribute to the core modules.
In the process of doing so, I noticed that there wasn't any resource that completely
described how to get started on your dev environment. This article documents
the steps that I took to get up and running. Hopefully it will be a helpful
resource for you too.

I should mention though that the following links helped me get started quickly
in certain aspects. So a big thanks to the authors! This article builds on these
foundations.

- [Unit Testing Ansible Modules](http://linuxsimba.com/unit_testing_ansible_modules_part_1)
- [Module Development Page](http://docs.ansible.com/ansible/developing_modules.html#testing-modules)


## Sidebar: If You See Any Errors in This Doc

Please don't hesitate to let me know via the comments below. Or if pull requests are
your kind of thing, the source for this website is on
[GitHub](https://github.com/relaxdiego/relaxdiego.github.com).

## Prerequisites

- Git branching basics. [Learn Git Branching](http://learngitbranching.js.org)
  is a great resource!

- Python 2.7

- Pip

- The following Python libraries:
    - paramiko
    - PyYAML
    - Jinja2
    - httplib2
    - six
    - nose
- Basic testing knowledge including mocking. If you're not too familiar with
  mocking, you can still follow along but I would encourage your to read up
  on it after you're done here as it will help make your tests more robust.

## Prep Your Ansible Repo

If you haven't already done so, fork the [Ansible](https://github.com/ansible/ansible)
repo to a GitHub account or organization you have access to. Afterwards, clone your
fork recursively (includes its submodules):

    $ git clone git@github.com:<your-github-account>/ansible.git --recursive

Then, add the upstream Ansible repo as 'upstream'

    $ cd ansible
    $ git remote add upstream https://github.com/ansible/ansible

After this, you now should have two remotes with 'origin' pointing to your
fork of the repo, and 'upstream' pointing to the upstream repo. You won't be
able to push upstream but you can fetch from it and then create pull requests
which is how it should be.

    $ git remote -v
    origin  git@github.com:<your-github-account>/ansible.git (fetch)
    origin  git@github.com:<your-github-account>/ansible.git (push)
    upstream        https://github.com/ansible/ansible (fetch)
    upstream        https://github.com/ansible/ansible (push)

I'm going to refer to your local clone the **ansible repo** from now on.


## Sidebar: Add a Few Git Aliases to Your Toolkit!

You don't have to do this but if you want to follow along with my
git commands, I'll be using these below so feel free to add these to
your `~/.gitconfig` under the aliases section:

    [alias]
    fa = fetch --all
    far = fetch --all --recurse-submodules
    t = log --graph --pretty=oneline --abbrev-commit --decorate --color
    ta = log --graph --pretty=oneline --abbrev-commit --decorate --color --all

With these, you'll gain the following `git` commands:

- `git ta` - See the history of your repo laid out in a tree
- `git t` - See just the tree of the current branch
- `git fa` - Fetch (but don't merge) the latest from all remotes
- `git far` - Like `git fa` but also includes those from git submodules


## Prep Your Core (or Extras) Modules Repo

Go to the core modules subdir

    $ cd lib/ansible/modules/core

You are now in a git submodule. If you run `git ta` here, you will
notice that the output is different from when you run the command one
level up. If you're not familiar with git submodules, it's really just
a separate git repo that's being referenced by a parent repo. when we
ran our `git clone` command earlier with the `--recursive` option, git
automatically cloned this repo as well.

If you view this repo's remotes, you will see:

    $ git remote -v
    origin        https://github.com/ansible/ansible-modules-core (fetch)
    origin        https://github.com/ansible/ansible-modules-core (push)


We want the naming of this repo's remotes to be consistent with that of
the parent repo. So let's rename origin to upstream:

    $ git remote rename origin upstream

Then, if you haven't already done so, create a fork of the
[ansible-modules-core](https://github.com/ansible/ansible-modules-core)
repo in the same account where you placed your fork of the Ansible repo.

Now, add that fork as a remote to your local repo and name it 'origin':

    $ git remote add origin git@github.com:<your-github-account>/ansible-modules-core.git

Your remotes list should now be:

    $ git remote -v
    origin  git@github.com:<your-github-account>/ansible-modules-core.git (fetch)
    origin  git@github.com:<your-github-account>/ansible-modules-core.git (push)
    upstream        https://github.com/ansible/ansible-modules-core (fetch)
    upstream        https://github.com/ansible/ansible-modules-core (push)

I'm going to refer to this local clone the **core modules repo** from
now on.

If you plan on contributing to the extras repo too, just repeat the same steps
above but replace core with extras.


## Prepare Your Environment For Local Development

While at the root dir of your **ansible repo**, run the following:

    $ source hacking/env-setup

This will prepare your current terminal session and prepend the current
ansible repo to your $PATH. Running `ansible --version` should get you
something similar to this:

    ansible 2.2.0 (devel e81f14ab48) last updated 2016/06/09 09:45:16 (GMT -700)
      lib/ansible/modules/core: (detached HEAD b37429f6ed) last updated 2016/06/10 09:08:18 (GMT -700)
      lib/ansible/modules/extras: (detached HEAD 93b59ba852) last updated 2016/06/09 09:45:29 (GMT -700)
      config file = 
      configured module search path = Default w/o overrides

Provided you didn't have any shims messing up your path, running
`which ansible` should now show the `<path to your local ansible repo>/bin/ansible`.
If not, you probably have some crazy shim-y stuff going on.

If you want to make this permanent, add the following to your `~/.bash_profile` or
`~/.bashrc`:

    source <path to ansible repo>/hacking/env-setup

## SIDEBAR: Keep Your Sanity. Use vim + tmux!

...well, if you're already familiar with these tools, that is.

To avoid having to cd up and down as I work on my modules, I use test.vim
with vim and split my tmux windows into the following panes:


[![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/af1952bf-d885-4f9f-8c19-7d62c1339aac.png)](http://www.relaxdiego.com/assets/images/ansible-vim-tmux.png)


<center>Click to expand</center>

1. The top pane is dedicated to Vim
2. Lower left pane is my local **ansible repo**
3. Lower middle pane is my local **core modules repo**
4. Lower right pane is for running my tests which I can quickly initiate
   from Vim thanks to test.vim by pressing `,s` (run nearest test),
   `,t` (run all tests in current file), and `,a` (run all tests)

I have my dotfiles project [here](https://github.com/relaxdiego/dotfiles). Yes
I still need to convert it to an Ansible playbook!


## Install The Ansible Module Validator (Linter)

It's like Flake8 for Ansible. Install it with:

    $ pip install git+https://github.com/sivel/ansible-testing.git#egg=ansible_testing

Try running it with:

    $ ansible-validate-modules lib/ansible/modules/core/cloud/amazon

It shouldn't output anything since the amazon core modules are compliant.
We will see it in a fouler mood later when we write a sample module.


## Enable Travis CI Validation

Enabling the repos you forked above in Travis CI will give you quick
feedback on where you messed up just by pushing your code to origin.
Best of all is that it's free! Sign up to travis with your GitHub
account if you haven't already done so then head on to your account
page and enable all the repos your forked above. If they're not listed,
click the Sync account button.

![](http://www.relaxdiego.com/assets/images/travis-ci-enable.png)

From now on, anytime you push to your repos, you'll get feedback from Travis.
No need to create pull requests just to know you messed up! I should note,
however, that Travis won't automatically run the tests you will write below
until [this pull request](https://github.com/ansible/ansible-modules-core/pull/3909)
is merged.


## Create Your First Module

Let's write a module for a fictitious cloud provider named Somebody's Computer.
First, in the **core modules repo**, let's create our module's subdir:

    $ mkdir cloud/somebodyscomputer
    $ touch cloud/somebodyscomputer/__init__.py

From now on, I'll refer to this as the **first module dir**.

Next, let's create a unit test directory under the test dir:

    $ mkdir test/unit/cloud/somebodyscomputer

## Let's kick the tires for a bit...

First, let's create a branch in the core modules repo to track one of the
branches upstream. I'm going to choose the devel branch for this example.

    $ git branch -t devel upstream/devel

This is just for convenience so that you'll have a local branch tracking
an upstream branch as you work. I WOULD NOT recommend working on this
branch directly though. Instead, create a topic branch from the HEAD of
devel and work there. Should new changes be added to upstream/devel, just
fetch those and rebase your topic branch as needed.

Let's create our topic branch:

    $ git checkout -b test_branch devel

Let's use the simple example from the [Module Development Page](http://docs.ansible.com/ansible/developing_modules.html#testing-modules)
just to get familiar with the terrain a bit. In your **first module dir**,
create a file called `timetest.py` with the following contents:

```python
#!/usr/bin/python

import datetime
import json

date = str(datetime.datetime.now())
print json.dumps({
    "time" : date
})
```

You just created your first module! At this point, we can create a
playbook that uses your timetest module and then execute it with
ansible-playbook. But why do that when the Ansible repo provides
a convenient script that allows you to bypass all that! So from
your **ansible repo**, run: 

    $ hacking/test-module -m <path to module dir>/timetest.py

This should give you an output similar to the following:

    * including generated source, if any, saving to: ~/.ansible_module_generated
    ***********************************
    RAW OUTPUT
    {"time": "2016-06-09 11:00:01.445100"}


    ***********************************
    PARSED OUTPUT
    {
        "time": "2016-06-09 11:00:01.445100"
    }

What just happened is that the test-module script executed your
module without loading all of ansible. This is a nice way to quickly
do a sort-of-end-to-end test of your module after you've written your
unit tests. I **would not** recommend using it exclusively as your testing
strategy. It's best used alongside unit tests, and `ansible-validate-modules`
which we'll use next.

Run:

    $ ansible-validate-modules <path to your first module dir>

This should get you the following errors:

    ============================================================================
    <path to first module dir>/timetest.py
    ============================================================================
    ERROR: No DOCUMENTATION provided
    ERROR: No EXAMPLES provided
    ERROR: Did not find a call to main
    ERROR: Did not find a module_utils import
    ERROR: GPLv3 license header not found

Ignore those errors for now while we're still kicking the tires.
Let's push our code up to origin. From your **core modules repo**, run:

    $ git add .
    $ git commit -m "Test commit"
    $ git push origin

This should automatically trigger Travis. Head on over to
`https://travis-ci.org/<your-account>/ansible-modules-core/branches` to see
the action. The tests will fail but that's alright because we're just familiarizing
ourselves with the workflow for now.



## Let's Write A Real(-ish) Module!

Let's undo the last commit because it sucks. From your **core modules repo**, run:

    $ git reset --soft HEAD^

Then delete timetest.py:

    $ rm cloud/somebodyscomputer/timetest.py

Next, let's install some Python packages needed by our tests. From your
**core modules repo**, run:

    $ pip install -r test-requirements.txt

## SIDEBAR: Here Be (Testing) Dragons!

I expect that you already know how to write good tests and mocks because I
don't have time to teach you that. If you don't, read up on some articles and
books about it and then come back here. You can still follow along and you might
make out a few things. But testing know-how will go a long way in these parts.

If you're confident with your mad testing skillz but your mocking-fu is a bit
rusty, I will have to ask you to read
[Mocking Objects in Python](http://www.relaxdiego.com/2014/04/mocking-objects-in-python.html).
It's a quick 5~6-minute read.



## Let's Write the Test First

We want our module to instantiate AnsibleModule and specify two arguments,
namely url and dest. So let's write our test to validate that.

First, since we're going to be using nose as our test framework, we have to
ensure that every subdirectory in the following path as an `__init__.py`
otherwise nose will not load our tests. Go ahead and make sure there's
that file in every directory in this path:

    test/unit/cloud/somebodysocomputer/


Next create
`<core modules repo>/test/unit/cloud/somebodyscomputer/test_firstmod.py`
with the following contents:


```python
import mock

from cloud.somebodyscomputer import firstmod


class TestFirstMod:

    @mock.patch("cloud.somebodyscomputer.firstmod.AnsibleModule", autospec=True)
    def test__main__success(self, ansible_mod_cls):
        firstmod.main()

        expected_arguments_spec = dict(
            url=dict(required=True),
            dest=dict(required=False, default="/tmp/firstmod")
        )
        assert(mock.call(argument_spec=expected_arguments_spec) ==
               ansible_mod_cls.call_args)
```

- **Line 8** - We're mocking AnsibleModule making sure autospec is True so that we
  don't create a false positive via fat-fingering arguments and methods. We also
  don't do anything else with the mock until we do our assertions because
  this test is only about checking if the module provided the correct values
  to argument_spec.
- **Line 16** - This is where we assert the call to our mock class to check
  if it called it properly.

Let's execute this test. From the **core modules repo**, run:

    $ nosetests --doctest-tests -v test/unit/cloud/somebodyscomputer/test_firstmod.py


This should get you an error because we haven't written our module yet:

    Failure: ImportError (cannot import name firstmod) ... ERROR

    ======================================================================
    ERROR: Failure: ImportError (cannot import name firstmod)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "/usr/local/var/pyenv/versions/2.7.10/lib/python2.7/site-packages/nose/loader.py", line 418, in loadTestsFromName
        addr.filename, addr.module)
      File "/usr/local/var/pyenv/versions/2.7.10/lib/python2.7/site-packages/nose/importer.py", line 47, in importFromPath
        return self.importFromDir(dir_path, fqname)
      File "/usr/local/var/pyenv/versions/2.7.10/lib/python2.7/site-packages/nose/importer.py", line 94, in importFromDir
        mod = load_module(part_fqname, fh, filename, desc)
      File "/Users/mmaglana/src/github.com/relaxdiego/ansible/lib/ansible/modules/core/test/unit/cloud/somebodyscomputer/test_firstmod.py",
     line 3, in <module>
        from cloud.somebodyscomputer import firstmod
    ImportError: cannot import name firstmod

    ----------------------------------------------------------------------
    Ran 1 test in 0.001s

    FAILED (errors=1)


## SIDEBAR: That's a Lot of Typing Just to Run One Test!

Well, if you set up your editor properly, you can run it with as few as
two keystrokes! Don't know how to do it, check out
[what I did](http://www.relaxdiego.com/2015/11/my-dev-setup-part-3.html).


## Let's Write Our First Code to Pass the Test

Create `<core modules repo>/cloud/somebodyscomputer/firstmod.py` with the following contents:


```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule


def main():
    AnsibleModule(
        argument_spec=dict(
            url=dict(required=True),
            dest=dict(required=False, default="/tmp/firstmod")
        )
    )

if __name__ == '__main__':
    main()
```

Run the test again to see it pass:

    ansible.modules.core.test.unit.cloud.somebodyscomputer.test_firstmod.TestFirstMod.test__main__success ... ok

    ----------------------------------------------------------------------
    Ran 1 test in 0.024s

    OK


## Now Let's Actually Write Useful Code!

Our module doesn't do much other than accept arguments, initialize a class,
then exit. Let's make it more useful. First, we update the test with additional
mocks and expectations. So let's replace the contents of our test with this:


```python
import mock

from cloud.somebodyscomputer import firstmod


class TestFirstMod:

    @mock.patch("cloud.somebodyscomputer.firstmod.write_data", autospec=True)
    @mock.patch("cloud.somebodyscomputer.firstmod.fetch_data", autospec=True)
    @mock.patch("cloud.somebodyscomputer.firstmod.AnsibleModule", autospec=True)
    def test__main__success(self, ansible_mod_cls, fetch_data, write_data):
        # Prepare mocks
        mod_obj = ansible_mod_cls.return_value
        args = {
            "url": "https://www.google.com",
            "dest": "/tmp/somelocation.txt"
        }
        mod_obj.params = args

        # Exercise code
        firstmod.main()

        # Assert call to AsnibleModule
        expected_arguments_spec = dict(
            url=dict(required=True),
            dest=dict(required=False, default="/tmp/firstmod")
        )
        assert(mock.call(argument_spec=expected_arguments_spec) ==
               ansible_mod_cls.call_args)

        # Assert call to fetch_data
        assert(mock.call(mod_obj, args["url"]) == fetch_data.call_args)

        # Assert call to write_data
        assert(mock.call(fetch_data.return_value, args["dest"]) ==
               write_data.call_args)

        # Assert call to mod_obj.exit_json
        expected_args = dict(
            msg="Retrieved the resource successfully",
            changed=True
        )
        assert(mock.call(**expected_args) == mod_obj.exit_json.call_args)

```

- **Line 8 and 9** - We're mocking two internal methods that we want
  our main method to call. I know some purists will be shocked at this saying
  that internal methods should not be mocked. My response is that sometimes
  you have to do this to avoid making your tests too complicated.
- **Line 13 to 18** - We're mocking the params attribute of the AnsibleModule
  instance since it's needed by our test subject.
- **Line 23 to 43** - We then check wether the method calls that we care about
  were called correctly by our test subject.
  
Running the above test should result in a failure, naturally. So let's write the
code to pass it. Your `firstmodule.py` should now look like this:


```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule


def main():
    mod = AnsibleModule(
        argument_spec=dict(
            url=dict(required=True),
            dest=dict(required=False, default="/tmp/firstmod")
        )
    )

    data = fetch_data(mod, mod.params["url"])
    write_data(data, mod.params["dest"])

    mod.exit_json(msg="Retrieved the resource successfully", changed=True)


def fetch_data(mod, url):
    raise NotImplementedError


def write_data(mod, data, dest):
    raise NotImplementedError

if __name__ == '__main__':
    main()
```

You'll notice that our `fetch_data` and `write_data` methods only raise an exception.
That's intended because we don't care about it yet at this point. Remember that we
mocked these methods in our test. Run the tests and see it pass.

## SIDEBAR: But If They're Mocked, Why Write The Methods At All?

That's because I patched them with the `autospec` argument set to `True` in the
test. When we do that, the test will throw an error saying that the method signatures
could not be found. This is a great way to protect ourselves from creating a mock
of a class or method signature that doesn't actually exist. Without this protection,
we'll create false positives everywhere in our test, rendering it useless.

## Let's Implement fetch_data()

We're going to make use of `StringIO` here to simulate the return value of
an ansible `mod_util` package which we'll have `fetch_data()` use. At the top of
your `test_firstmod.py`, add this line:

```python
from StringIO import StringIO
```

Then let's add this to the bottom of the same file:

```python
# fetch_data test
    @mock.patch("cloud.somebodyscomputer.firstmod.fetch_url", autospec=True)
    @mock.patch("cloud.somebodyscomputer.firstmod.AnsibleModule", autospec=True)
    def test__fetch_data__success(self, ansible_mod_cls, fetch_url):
        # Mock objects
        mod_obj = ansible_mod_cls.return_value
        url = "https://www.google.com"
        
        html = "<html><head></head><body></body></html>"
        data = StringIO(html)
        info = {'status': 200}
        fetch_url.return_value = (data, info)

        # Exercise the code
        returned_value = firstmod.fetch_data(mod_obj, url)

        # Assert the results
        expected_args = dict(module=mod_obj, url=url)
        assert(mock.call(**expected_args) == fetch_url.call_args)

        assert(html == returned_value)
```

- **Line 2** - We patch ansible's fetch_url utility method making sure to set
  `autospec` to `True`
- **Lines 6 and 7** - We prepare the arguments to pass to our test subject
- **Lines 9 to 12** - We mock out `fetch_url`.
- **Lines 18 to 21** - We then check wether our test subject made the correct
  call to `fetch_url` and then also check that it returned the correct value.
  
Run the test and see it fail. Let's now write the code that makes it pass.
First, add this near the top of your `firstmod.py` right after the first
import line:

    from ansible.module_utils.urls import fetch_url

Then, modify your `fetch_data` method to look like this:

```python
def fetch_data(mod, url):
    data, info = fetch_url(module=mod, url=url)
    return data.read()
```

Run the test again. BOOM!


## Let's Implement write_data()

Add the following test to the bottom of `test_firstmod.py`:

```python
# write_data test
    def test__write_data__success(self):
        html = "<html><head></head><body></body></html>"
        dest = "/tmp/somelocation.txt"

        o_open = "cloud.somebodyscomputer.firstmod.open"
        m_open = mock.mock_open()
        with mock.patch(o_open, m_open, create=True):
            firstmod.write_data(html, dest)

        assert(mock.call(dest, "w") == m_open.mock_calls[0])
        assert(mock.call().write(html) == m_open.mock_calls[2])
```

- **Lines 6 to 8** - We expect the test subject to use the builtin
  `open` method as a context manager when it writes the file. This
  is the de facto way of mocking it.
- **Line 11** - We check that the test subject opened the destination
  file for writing.
- **Line 12** - We check that the test subject actually wrote to the
  destination file.

You know the drill. Run to fail. Now write the code to pass it:

```python
def write_data(data, dest):
    with open(dest, 'w') as dest:
        dest.write(data)
```

Run the test again. BOOM!

## Almost There!

Let's run the linter against our module:

    $ ansible-validate-modules <path to module dir>

That should list a few errors about documentation, examples,
and the GPLv3 license header. Let's fix that by adding the following
between the import declarations and the `main()` method:

```python
# This file is part of Ansible
#
# Ansible is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Ansible is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with Ansible.  If not, see <http://www.gnu.org/licenses/>.

DOCUMENTATION = '''
---
module: firsmod
short_description: Downloads stuff from the interwebs
description:
    - Downloads stuff
    - Saves said stuff
version_added: "2.2"
options:
  url:
    description:
      - The location of the stuff to download
    required: false
    default: null
  dest:
    description:
      - Where to save the stuff
    required: false
    default: /tmp/firstmod

author:
    - "Your Name Here (@yourgithubusernamehere)"
'''

RETURN = '''
msg:
    description: Just returns a friendly message
    returned: always
    type: string
    sample: Hi there!
'''

EXAMPLES = '''
# Just download it
- firstmod:
    url: https://www.google.com

# Download then save to your home dir
- firstmod:
    url: https://www.relaxdiego.com
    dest: ~/relaxdiego.com.txt
'''
```


For an explanation of these strings, see Ansible's [Developing Modules Page](https://github.com/ansible/ansible/blob/devel/docsite/rst/developing_modules.rst#example).

Run the linter again. BOOM!

## Test The Module Manually With Arguments This Time

We can write a playbook that uses our module now if we want to
but I'll leave you to do that on your own time. For now, let's
run it using the `test-module` script that comes with the
**ansible repo**. Run the following:

    $ <path to ansible repo>/hacking/test-module -m <path to first module dir>/firstmod.py -a "url=https://www.google.com dest=/tmp/ansibletest.txt"

Next, run the following to see what it downloaded:

    $ cat /tmp/ansibletest.txt

## Time To Push to Origin!

Run the following from the core modules repo:

    $ git add .
    $ git commit -m "First module"
    $ git push -f origin

Note the `-f` in the last command. This is because we need to
overwrite the previous commit that we pushed earlier. I wouldn't
recommend this for shared branches but is perfectly safe for
unmerged topic branches where you're the only person working on it.

Head on over to Travis to see the action!

Now if this were a real module, your next step would be to create
a pull request from your branch to the upstream branch. But since
this is just a demo project, grab yourself a beer and bask in your
awesomeness!

## SIDEBAR: Speaking of submitting code upstream...

Always make sure that your topic branch is based off of the HEAD
of the parent branch. In our case above, we created `test_branch` off
of `upstream/devel`. As we're working on `test_branch`, new commits are
merged to `upstream/devel`. It is, therefore, a good idea to do this
regularly (while in a topic branch of **core module repo**):

    $ git add .
    $ git commit
    $ git fa
    $ git rebase upstream/devel

Keep doing this until your topic branch is accepted and merged
upstream. That should put your topic branch ahead of `upstream/devel`
and avoid any merge conflicts down the line. If you do this regularly
and do encounter a merge conflict, it will be easier to fix now
than later.

## What, You're Still Here??!

Yup, that's the end of it for now. Post in the comments or [file a bug](https://github.com/relaxdiego/relaxdiego.github.com/issues/new)
if you find any errors or have any questions. If you read this
in one sitting, you deserve another bottle of beer!

(This article originally appeared in my blog at http://www.relaxdiego.com/)