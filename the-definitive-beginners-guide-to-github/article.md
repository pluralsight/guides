I'm going to give beginners an all inclusive guide to Github, however this is not a guide to Git in and of itself. I will be doing a full article on Git soon... In the meantime I will put a little bit at the end of this article for people to get a small taste for Git. For the rest of you this is going to be an overall guide to the inner workings of Github in the way I understand and see it to work.

> FYI: If you are used to Github and experienced with it, then most of this guide might be redundant for you. However there also might be a few gems you didn't know about!

# Introduction
First of all, what is Github? Github is the largest single point or hub of open source and world wide development. It uses a source code versioning system called `Git`, which is very powerful and lends itself well to open source communities. On Github you will find all types of hidden gems that can help you become a better developer. I spend at least a few minutes a day browsing and looking for new projects, inspecting their code, and learning from the community. 

> It is by far (In My Honest Opinion) the easiest way to manage your source code, project, documentation, and more.

# Signing Up
To get started you will need to sign up. To do this go to the [Homepage](https://github.com) and sign up. Make sure to choose your username wisely, and try not to use any special characters such as dashes or underscores. Readability is key. This username will be used in the links and navigation of all your projects going forward. 

# Your Profile
Your profile holds a lot of information about you, your activity, and the projects you work on. 

See below an example of my profile:
![Shadowcodex Github Profile](http://i.imgur.com/evh20PN.png)

There are three main parts to your profile. 

1. Personal Information
2. Repository Information
3. Contribution Information

Your personal information is your avatar, contact information, followers, stars, and organizations in which you participate. This is very simple, and you can see the example of my personal information section below:

![Shadowcodex Personal Information](http://i.imgur.com/ENfEcA9.png)

> Quick Tip: Your profile image should be Between 1000x1000 and 500x500 pixels to look good. 

On the other hand you have the repository information. The repository information will contain a list of all repositories you have interacted with. They are broken down into two categories `Personal` and `Contributed to`. Personal repositories will be all of the projects and source code that you have setup on your own profile. If you do anything with another organization's project repository (repo), then it will show up under the contributed to section. See my repository area example:

![Shadowcodex Repository Area](http://i.imgur.com/CRzWXZl.png)

Contributions are often seen as the gauge of activity on Github. Below your repository section you will see a blank graph (Blank to you if you are new). Each square will turn darker green with more contributions for that day. I haven't been very active this year, and have only picked back up in November so mine is very blank.

Here is an example of my contribution graph for this year:

![Shadowcodex Contribution Graph](http://i.imgur.com/kyAstAf.png)

Here is an example of a really active contribution graph: [Hitman666](https://github.com/Hitman666) A fellow Pluralsight Author.

![Hitman666 Contribution Graph](http://i.imgur.com/axURlx9.png)

Kudos to Hitman666 for that continuous block of green.

> Quick Tip: If your contributions are not showing when you are pushing commits then you should make sure your email is setup in your git config on your development machine. If it doesn't match the email in your Github profile you won't get the credit for it.

Below that section you will see a contribution list. By clicking on one of the green blocks on your profile (Or someone else's profile) you can filter and sort through the contributions for that day. It is a really good way to show all you have done for that day. It also allows you to go to that specific contribution to inspect, comment or sometimes modify it.

# Finding Projects
Github has made it extremely easy to find projects. There are many reasons you will want to search for projects on Github. The biggest reasons are to find a tool / code to use in your personal / team project, find a new project to contribute to, find an experienced and mature project to learn concepts from (Unit testing, documentation, etc..) or just to peruse to find cool stuff.

To start, look for the search bar at the top of the screen. It has two modes. Global and Repository.

Global looks like this:

![Global Search](http://i.imgur.com/ybSyVvm.png)

Repository looks like this:

![Repository Search](http://i.imgur.com/JxnPivh.png)

When you are on global you can search by anything (Just like google). So let's search JavaScript because that's my favorite language. (You haters are gonna to hate...). 

The first thing it is going to pull up is repo's with the name Javascript in them. This doesn't help me a lot, because I am searching for something cool that has Javascript in it. But it is a good stopping point to go over the sections of the search screen with you.

The search we just did looks like this:

![Github Javascript Search](http://i.imgur.com/1XOFO88.png)

This too has several sections like the profile, so I'll go over them quickly. On the left you can see three immediate sections. Two of these help refine your search, and one gives you more information. The first one shown here is your search type. You can use it to tell the search engine if you are looking for repositories, text in code, text in issues, or a user. In this case we are looking for a repo so we will just leave it where it is.

![Search Type](http://i.imgur.com/7SIU8D9.png)

The next section is a language selector. This helps you sort to only projects that contain the following languages. Since we searched JavaScript, it is obviously at the top with 82k repositories. I'll go ahead and select it to refine our search a little.

![Language Refine](http://i.imgur.com/nm9w4Bf.png)

Finally on the left is the advanced search. You can play with that later if you want, but I wanted to point out the cheat sheet. It can be very useful in refining your search. I see that there is a cheat to showing repositories with a certain amount of stars. I'll go ahead and refine my search to have only repos with over 1000 stars. (Those are really popular ones).

![Github Search Cheatsheet](http://i.imgur.com/AV4CxO5.png)

Here are the new search results.

![Search Refine](http://i.imgur.com/AseYU49.png)

As you can see the new search is much more refined. Went from about 130,000 repositories to just 497. Also in the screenshot above you can see the sort which is the next thing I wanted to talk about. There are a lot of different sorts, but I'm looking for an active project. I'm going to go ahead and sort by `Recently Updated`.

Below is the final outcome of my search, and I am going to choose [Highlight.js](https://github.com/isagalaev/highlight.js) for the next section of the article.

![Final Search Results](http://i.imgur.com/0oocXJJ.png)

# The Project Page
In the previous section I went over the search feature for finding projects on Github. During that section we ended up landing at a repository for HighlightJS. Here is what the repository page looks like:

![HighlightJS Repository](http://i.imgur.com/gZHw1Xb.png)

Github has put a lot of work into the `User Interface` and `User Experience` of this page. I'm going to only cover the main parts of the homepage of a repository, and what some of the links are. But we will get into actually using this page when we talk about managing your project.

There are 4 main parts of the Github Project page. The top part with links and menus, the Project Information area, the Source Code area, and finally the `README.md` at the bottom of the screen.

In the header you will find links to Code, Issues, Pull Requests, Pulse, and Graphs. Each of these has it's own purpose. 

* *Code* : The actual source code of the project
* *Issues* : Feature-Suggestions, bugs, enhancements, and conversation about the project.
* *Pull Requests* : Code that has been submitted to be part of the project.
* *Pulse* : Analytics and Overview of the activity of the project.
* *Graphs* : Deeper analytics and information about all sorts of project details.
 
![Repo Menu](http://i.imgur.com/ydM9icm.png)

To the right of that you will find three buttons you should get very familiar with. Watch, Star, and Fork.

* *Watch* : This will allow you to receive notifications of all the activity on a repo. Be careful on watching highly active repositories or you will get an email inbox full of messages.
* *Star* : This is the `like`, `love`, `favorite` of Github. Marking this will bookmark this repository under your Stars area in your Personal Profile. Remember that under your avatar (Scroll up to see that image again).
* *Fork* : In my opinion this is the most important button on the page. Fork is a concept of Git and source code contribution. If you are going to contribute to a project then you will have to fork it and make your edits in your own version of the code. Github Makes this easy by doing it for you when you press this button.

![Watch, Star, and Fork](http://i.imgur.com/7cFlUck.png)

Below those three important buttons you will have the project information area. This area shows how many commits, branches, releases, and contributors are on the project.

* *Commits* : These are the base behind the Git Versioning system. Instead of pushing all of the code to the server every time you make changes, the system only stores what those changes are. This makes it easy to roll back, and see what someone has actually done with the code. A commit is a package (Boxed up and ready to go) of all the changes done up to that point since the last commit.
* *Branches* : When working on a specific idea, you want to leave your main code alone. Here you can copy your code over to a new "Branch" and there you can make your changes without modifying the original. When you feel like you have good code, have tested it, and want to move it back to the main branch you can do that with a pull request. (More on that in the coming sections).
* *Releases* : This is a version release, and let Github know to mark this commit as full package. People can go here and download the code as it was on each release.
* *Contributors* : Contributors are people who have worked and helped maintain a project. 

![Project Information Bar](http://i.imgur.com/RZns4EX.png)

If you notice the colored part underneath that bar, you can click on it. This will reveal statistics on the distribution of programming languages used in the program.

![Language Distribution](http://i.imgur.com/k2iGJx1.png)

The next section allows you to traverse through the code in the project. View it, edit it, etc... Though if you edit someone else's code, note that it will fork it to your own repository for editing. 

![File Listing](http://i.imgur.com/HN56K22.png)

If you were to click on a file it would take you into the file editor. I clicked on AUTHORS.en.txt to show this to you. It shows who submitted the latest commit, a line of contributors, how many lines of code it is, the file size, and much more information.

![Edit File Screen](http://i.imgur.com/9StqBnX.png)

You will also notice a few buttons on the top right of the editor. I'll quickly describe each one for you.

* *Raw* : This will open a new window and show the raw contents of the file. Allowing you to select it all to copy, etc...
* *Blame* : This is another view of the code. For each line of the code, it shows all the recent commits on the left, with the code on the right. 
* *History* : This shows in chronological order the changes made to a file.
* *Open in GH Desktop* : This will download the file and open it in Github's desktop source control software. 
* *Edit* : If this is your file it will open the file for you to edit and update. If it is someone else's file it will open it up in a fork for you to edit.
* *Delete* : Again if this is your file it will just delete the file. If it is someone else's file it will fork the project to your own repository and delete this file from it.

Back on the repo main page, you can see the section below the files which is called the README. This is the main documentation point for all repositories on Github. You use this to describe your project and more. That leads us into the next section. Creating a project and Managing it.

# Creating a Project
It's really easy to create a project. Just click the plus sign at the top of any page and choose `New repository`. 

Click This: 

![New repo](http://i.imgur.com/kXgOirS.png)

On the next page just fill out the details. Click `initialize this repository with a README` and most of the time I choose the MIT License. If you click the info sign (next to the license), Github can tell you all about the different types of licenses available to chose from. It's important to note that due to `Github's Terms of Service` you have to use an open source license for public repositories. After you have filled it out, you can click `Create repository`. Here is the repository I am making for this article.

![Create Repo Screen](http://i.imgur.com/E4tshgQ.png)

# Managing a Project
Now that I have created a repository, I'll want to manage it. There is a lot that goes with managing a Github repository. But don't fret I'm going to walk you through most of it. 

## Readme

The first thing you should do is set up a README.md. This is located in the root directory of your project and can be edited in the browser pretty easily. The readme uses what is called `Markdown Syntax` and you can find all the styling information about that here: [Guide To Markdown](https://help.github.com/articles/markdown-basics/)

> Quick Tip: The Readme should have basic information about your project. How to download it and use it, as well as other small task to be performed. You can see a good example of a Readme [by clicking here](https://github.com/Unrestricted-Coding/realtime-chat-RethinkDB)

For this readme I am going to just describe this repository. Here is my markdown for this.

```markdown
# The-Definitive-Guide-To-Github
This is the repo used in an article published to Pluralsight.

I am making this repository to allow others to view it after reading the article. It also allows me to do screenshots and perform actions along side writing the article. This is good practice to make sure I don't miss anything.

You can find the article at [This link](http://insertarticlelinkehere)
```
There is a lot of stuff that you can do while editing this file. You can preview the changes and also change the name. When you are done editing you need to name your commit (Remember you are pushing changes, so this will be packaged as a commit and put in your repository) and a description of what you changed. You can see what I have done here: 

![Editing Readme](http://i.imgur.com/mdlwgcr.png)

I'll save that and I'll have an updated readme page on the repository.

![Updated Readme REPO](http://i.imgur.com/H95ovWQ.png)

## Issues

Issues are probably the second most important part of Github. Issues make managing bugs, feature requests, and other things a sinch.

![The Issues Page](http://i.imgur.com/vpFXljb.png)

> Quick Tip: Opened issues are GREEN and closed issues are RED...

To create a new issue just hit that big green `New issue` button up there. Give your issue a title, and then describe what the issue is. It can be anything from a `bug`, an `enhancement`, a call for `help wanted`, or even just a `question`. What I just described are Labels. And you can select labels on the right. You can also select a milestone if you have created one, or assign someone. Please note that some of these actions require you to be an owner of a repository. Once you are done you can click `Submit new issue`.

![Submit Issue](http://i.imgur.com/WJPVHC8.png)

On the issue page you can see all details about an issue. Including all the fields we just populated when submitting the issue. You can link issues to each other using the `#issuenumber` and link to commits using the `commit hash`. This is where you will have the discussion of your issue for further conversations, up until it is closed.

![Issue Page](http://i.imgur.com/lNDrhb2.png)

For beginners, you are probably good with the default labels so I am not going to go over that. Here is what an active issue board looks like...

![Active issues](http://i.imgur.com/RyzJX9F.png)

## Milestones
You can access milestones within the issues area. These are a great way to segment and plan your project. You set up a name, due date, and then issues can be assigned to milestones. Allowing you to store and manage issues easily. Here is what an active milestone page looks like: 

![Active Milestones](http://i.imgur.com/pDfv1bj.png)

## Wiki
There are really many ways of storing your documentation and a wiki is just one of those ways. (You can also just store markdown files in your repository, or use github pages). 

Wiki pages are just a separate area to store your documentation in markdown format. It separates your code from your documentation. Depending on your personal preferences, that may be a good or bad thing.

> Quick Tip: Most new projects are using github pages for documentation because you can use documentation generators like Mr-Doc to generate the html pages from comments in your source code. It's free so why not?

The wiki can be a quick way to get your documentation started. However I prefer to use Github Pages.

## Github Pages
Github Pages is one of the newer additions to the Github platform. It allows you to serve static websites for free. Yes I said for free. It's one of the best kept secrets that isn't a secret anymore. I've personally dropped all of my paid hosting and moved everything to github pages. 

The system uses a software called Jekyll to generate and maintain the static files, and that system is pretty easy to use. You can learn about Jekyll over [at it's main site](http://jekyllrb.com). Think of it like wordpress for the command line.

> Learn More: To learn more about Github Pages and all the complexities with it please check out their site at: [Github Pages Site](https://pages.github.com) and their help article [github pages help article](https://help.github.com/pages)

To use Github Pages all you have to do is create a new Branch in your repository called `gh-pages`. Drop your static web files (html, css, js, images, etc..) into the branch and it will be served over a url like : `http://your-username.github.io/repository-name`. It's as simple as that!

> Quick Tip: Github lets you use your own domain name for free, which is really awesome. Use that to host your full site at a domain name of your choosing.

## Building a Community
I'd guess that more than half of the repositories (Probably 8 out of 10) on Github are open source projects. Open source projects tend to work well for communities to buy into and help build. Because of that you will want to provide ways for your community to grow. Here are just a few of those ways.

### Integrate With [Slack](http://slack.com)
Slack is probably one of the best communication tools on the market right now. It's free to use and setup and with some well programmed APIs it can help build up a great community. For example by clicking on this [link](https://uc-slack.herokuapp.com) you can fill out a form and get invited to my Slack Community. 

I prefer to use Slack because it can be used for multiple projects, has a mobile app and is very simple to use. 

You can also integrate Slack with GitHub by using a service called "If This Then That" located at [IFTTT](http://ifttt.com). IFTTT allows you to build triggers. I have a lot of them setup for my team's Slack such that, when A new issue gets posted to any of my GitHub repositories, it automatically notifies people through Slack.

### Integrate With [Trello](http://trello.com)
Trello is a good tool to help keep your project organized. You can use it to organize `cards` into `categories`. This could be a new feature in the `In Progress` category. I have used an IFTTT flow to post new issues to a Trello Board where I can then work on them and move them through my task management workflow.

### Integrate With [Gitter](http://gitter.im)
Gitter is an alternative to using slack. I have not used it much, however I see a lot of repositories that do. One thing to keep in mind is that, if you work behind a corporate firewall they may block ".im" domain names.

# Contributing to Projects
This is the strongest part of GitHub in my opinion. I stated earlier that issues feature was #2 and this by far is #1. The concept of Github and Git in and of itself lends well to collaboration. For the rest of this article I will be talking about contributions and how to use Git. 

## Fork
Before you get started on helping with a project, you should first check with the owner of the repository for things that has to be done. Sure, you can add your ideas into the mix, but if there are major things that has to be done, then you need to understand that. 

After you decide you are going to contribute and have an idea of what you want to do, you will want to fork the repository. This will copy the repository to your own personal repository. Here you can clone the repository (See below about cloning), make your changes, push them back to your repository. You can do this as long as you want, but keep in mind that the repository that you forked from will keep on changing. Make sure you keep your code synced with changes that are happening with the main repository.

## Pull Request
After you have finished making the changes to your repository, you have tested them, and are confident they are ready then you need to submit a Pull request. I'll go ahead and make a small change to a Pluralsight Tutorial and submit a pull request so you can see how that looks.

You can see below that when I clicked to edit an article in the repository that Github forked it for me. Now when I save it I'll be all ready to submit a pull request.

![Forked an edit](http://i.imgur.com/Ak4p76w.png)

I'm going to go through and update code blocks in this file to have syntax highlighting. It's just a small pet peave of mine that I fix for Prateek (The guy who runs the tutorials site here).

After I am done editing I'll add a title and description to my edit and click `propose file change`. 

![Propose File Change](http://i.imgur.com/eiyX6gO.png)

After I click that it will move me to a comparing changes page where you can see that I changed 1 file and added 15 new lines of code. You can also see that I am going to be submitting a merge request (Pull request) from my head fork of `shadowcodex/tutorials` to the base fork of `pluralsight/tutorials`. I will hit the green `Create pull request` button and submit it.

![Submit Pull Request](http://i.imgur.com/oAU7gEl.png)

Next it will ask me to title and comment my pull request. Github was nice enough to prefill these from my proposed file changes area. I'll just go ahead and say that is good enough and click the green `Create pull request` button to finish the submission.

![Complete Pull Submission](http://i.imgur.com/z9l4uXM.png)

Now I am all done with my contribution. You can see in the image below that Prateek can now review the pull request. If he likes the changes he can go ahead and approve it and automatically merge the new code into the repository. With that I will have made a meaningful contribution to the pluralsight/tutorials repository.

![Merge Pull Request](http://i.imgur.com/TJZaSjN.png)

# Basic Introduction to Git
Now let's go over a few details about Git itself. What is Git? As we have discussed Git is a source code control system. It has been designed to work for everything from small to large scale projects. It's fast, easy, and most of all efficient.

To get a full understanding of git you should run over and read about it here: [Git-SCM](https://git-scm.com/documentation)... But basically the concept you are going to follow for 90% of the time is:

1. Clone
2. Make Changes
3. Commit
4. Push Commit to Server

or

1. Fork
2. Clone 
3. Make Changes
4. Commit
5. Push Commit to Server
6. Submit Pull Request
 

## Cloning
When you create your project on Github you will notice that on it's home page there is a link that looks like this: 

![Clone Link](http://i.imgur.com/7E3BKA2.png)

Once you have git installed on your local machine all you have to do to get this code onto your machine is run the following command using that link.

```shell
$ git clone https://your-github-link
```

This will copy the entire master branch (Or whatever branch you have set as default) to your local machine.

## Commit
Once you have cloned your repository you are free to make changes to the files. After you are done, you will want to push the changes back to Github. To do this we are going to make a commit which will store all the changes into a package ready to be pushed to the server. The following commands will add all the changes to a commit for you. 

> Quick Tip: Make sure you are in the root directory of your project before running these commands.

The first command adds all the files to your stage and prepares to commit them.
```shell
$ git add .
```

If you have deleted files then you may get an error when running this command. That's ok. Just run the following command instead.

```shell
$ git add . --all
```

You are now ready to make your commit. You can do that easily with the following command.

```shell
$ git commit -m 'enter your commit message here describing what you did'
```

Your commit should now be ready to go! 

>Pro Tip: You can have as many commits as you want on your local machine. I do commits when I finish major parts of the project. Once you do a push, it will push all the commits that aren't synced to the server.

## Push
Now that you have your commit ready then you will want to push it to the server. That can be easily done with the following command.

```shell
$ git push
```
It will ask you for your username and password. Once you have entered those correctly it will push all commits to the server. You're done!

## Pull
If you make changes on the remote repository then you will want to get those changes into your local repository. (Assuming you have already cloned it). You can do this with the pull command. Running the following command will pull any new commits on the server to your local machine.

```shell
$ git pull
```
## Checkout
If you have created a branch to work with on Github, you can easily switch to that branch on your local machine by running the following command:

```shell
$ git checkout your-branch-name
```

## Pull Request
I recommend that beginners do all pull requests on the website. It makes it easier and more fool-proof.

# Wrap Up
With that, I wrap up the article. It's been a great time, and I hope you have enjoyed learning all about Github. I look forward to seeing you on there. Just follow me, join my [Slack](https://uc-slack.herokuapp.com) and we will make some awesome things!

### About The Author:

![Shannon Duncan](https://pbs.twimg.com/profile_images/672481536826937344/GeAx6xl4_200x200.jpg) 

Shannon Duncan is the Founder and Author at [Unrestricted Coding](http://unrestrictedcoding.com). He mentors and teaches people of all ages on how to code and enjoy the art of programming. Find him on [Twitter](https://twitter.com/bikemybodyback), [LinkedIn](https://www.linkedin.com/in/jsduncan98), and [GitHub](https://github.com/shadowcodex).
