This entry is motivated by a tough week.

That's right, a week of challenges related to [scm] [scmlink] (which is not there the most relaxing thing in the world). A week full of failed attempts to [ `cherry-pick` git] [cherrypick] s and other git things, I probably would not use if the code was being managed" right ".

Now, no longer weeping and trying to look on the positive side, I learned some cool little things that can be useful to me in the future, and why not, for you :)

The idea here is not to teach git nor present the basic commands or something like that. The idea is to show some tricks and cool git tricks, you'll want to be known before.

So here we go ...


### Alias (co > checkout)
You can create [alias] [alias] for any git command.
You just need to set the alias to the command you want, like this:

```
$ git config --global alias.st status 
```

Just replace the **alias** with the name that you want and put the command in the front.

It is better to type "git co" doque "git checkout" right?
Just create the alias! I've seen people * (i) * using an alias * g * pro * command git *. It is much haha laziness. This would give you a kind thing:

```
$ g co
```

> It is a little different, but over time you earn enough in productivity.


### Autocomplete no shell

Use the git command line, and thank God the git autocomplete was always enabled here. If you use git shell by ruWindows, the autocomplete should already be set.

If the autocomplete is not enabled on your terminal, you can download the autocomplete script git [this link](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash).

After downloading, copy the file to the home directory, and add to your file *.bashrc* the line above:

```
source ~/git-completion.bash
```

Ready. Now when you enter a git command and press *tab*, it should display to you the commands:

```
$ git co<tab>
```

```
commit config
```

### Checkout e reset on files

You may have used these two commands: [checkout] [checkout] and *[reset][reset]*, to work with branches and commits.

What you may not know is that you can also use the *checkout* and *reset* files.

Quando usamos o *git reset* em um arquivo, o git **atualiza** a área de stage (staging area), fazendo com que um determinado arquivo **volte** ao estado em que se encontrava em um commit anterior qualquer.

When we use the *git reset* on a file in git ** update** the stage area (staging area), causing a particular file **back ** to the state it was in a previous commit.

This means we can remove the file *"jedi.js"* the stage area, and cause it to be equal to its version in *HEAD* with the command:

```
$ git reset HEAD jedi.js
```

The result of the operation, the file is outside the stage area and with the same content, present in the latest commit.

![Reset File](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/resetfile.png "git reset file")

Pretty cool!
Now let's take a look at **`checkout`**.

Good old *git checkout* we use to switch between branches, when used with a path to a file does git point to that file. Without updating the current branch.

```
$ git checkout HEAD jedi.js
```

Now our archive **"jedi.html"** is equal to this file *HEAD*, and the new version of the file is not in the stage area.

![Checkout File](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/checkoutfile.png "git checkout file")


> Save it somewhere, really. You will need one day.


Thus, you can only point to the version of a file to another branch (or the same), and make changes to then immediately add the modified file the staging area with the `git add .`.

### Stash, to be happy

The command `git stash`, picks **all** the stage area and save in a separate place, clearing your stage area.

That's really cool, because that way you can save a set of changes you have made but not want to commit for any reason yet.

```
$ git stash
```

In particular, I use more [stash] [stash] when I need to *git pull* and do not want the changes in my stage area generate conflict with the downloaded files. 

After creating your stash with the command `git stash`, you need to apply your stash. That is, add your stage area all the changes you put in the stash. To do this, simply use:

```
$ git stash apply
```

> You can have more than one stash. They will **always** applied in the order "first in, last out" if you do not specify the name of the stash right after *apply*.


### Amend, to redeem

f you have made a commit, and then remembered that he forgot to add a modification to a file (it happens), use the `--amend`.

Let's see:

![Commit amend](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/amend.png "git commit amend")

Here the parameter * - no-edit * Save your skin, making the commit message in which you are doing [*amend*] [amend], remain the same.

### Diffs between commits

To see the changes of the last commit, you can use:
```
$ git log --stat
```
This command will show the files and the number of lines added and removed by file in each commit.
To see what exactly was changed in a commit, use the [command diff] [diff].

To see the difference between two commits with [SHA] [sha] s of commits in hand (0da94be and 59ff30c), use:

```
$ git diff 0da94be 59ff30c
```

If you use GitHub, you may better see the differences there.
[+ see how][githubdiff]

### That's it folks !

There are some other tricks and cool stuff that git offers but pro post not get big, I chose the one I like.

If you think absurd the post does not have a legal tip you know, comments there for us :) 

Until next **o/**



[scmlink]:      		https://en.wikipedia.org/wiki/Version_control
[cherrypick]: 	 http://imasters.com.br/artigo/24442/desenvolvimento/dica-git-da-semana-cherry-picking/
[alias]: https://git-scm.com/book/tr/v2/Git-Basics-Git-Aliases
[checkout]: https://www.atlassian.com/git/tutorials/undoing-changes/git-checkout
[reset]: https://www.atlassian.com/git/tutorials/undoing-changes/git-checkout
[stash]: https://git-scm.com/book/pt-br/v1/Ferramentas-do-Git-Fazendo-Stash
[amend]: https://git-scm.com/book/pt-br/v1/Git-Essencial-Desfazendo-Coisas
[diff]: https://git-scm.com/docs/git-diff
[sha]: https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
[githubdiff]: https://help.github.com/articles/comparing-commits-across-time/