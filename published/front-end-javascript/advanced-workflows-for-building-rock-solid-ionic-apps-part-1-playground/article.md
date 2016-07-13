![Advanced workflows for building rock-solid Ionic apps. Part 1: Playground](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/blog-01.png)

At [M-Way Solutions](http://www.mwaysolutions.com/) we've been building large-scale enterprise-level apps with [Ionic](http://ionicframework.com/) for the past two years. The challenges we faced sparked the creation of [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), a powerful open-source collection of development tools alongside the [Ionic CLI](http://ionicframework.com/docs/cli/), that put the fruits of our extensive experience right at the end of your fingertips.

Are you looking for a single tool that you can use to quickly create compelling prototypes for impressing your customers *AND* that later scales with complex project requirements like testing, quality assurance, and continuous integration? Then look no further and prepare yourself for a thrilling ride through our generator's Building-apps-with-Ionic adventure island!

### Introduction
First stop is at the ancient ruins of Pre-Ionic-mobile-app-development: We've been developing mobile apps long before Ionic made it into our development stack along with Angular. We consistently explored different combinations of MV* frameworks and mobile frameworks throughout the years, we even developed our own mobile framework for quite some time but Angular & Ionic [kept consistently impressing us](http://blog.mwaysolutions.com/2015/09/10/generator-m-ionic-html5-mobile-app-development-evolved/). Focussing on Angular and Ionic allowed us to tackle many of the other challenges that app development, our business and enterprise customers pose.

These challenges led to the genesis of [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic) and catapulted us straight into Ionic-mobile-development-fun-land. The generator now is a valuable contribution to our development process and company culture and addresses the following aspects of mobile app development:

- **Provide useful workflows**
 - for development, testing, quality assurance, building, continuous integration
 - for complex project requirements like managing different sets and versions of APIs, app icons and splash screens
- **Embed nicely into ecosystem**
  - integrates nicely into different solutions like [Ionic Platform](https://ionic.io/platform) using the [Ionic CLI](http://ionicframework.com/docs/cli/) and our own [Relution](https://www.relution.io/en/)
  - use technology stack many developers already know: [Git](https://git-scm.com/), [Yeoman](http://yeoman.io/), [Bower](http://bower.io/), [Gulp](http://gulpjs.com/), [Cordova](https://cordova.apache.org/), ...
- **Standardize project setup**
  - tame and wire together an ever-changing and **complex frontend technology stack**
  - give a default **project & file structure**
  - come with **sensible default configurations** for development tools like [Git](https://git-scm.com/), [ESLint](http://eslint.org/) and others
  - is still easily modified to suit **different project requirements**
  - make app development more **approachable** for newcomers
  - simplify **project handovers**

So it now stars as the **one go-to starting point** for ourselves and a growing community when it comes to mobile app development with [Ionic](http://ionicframework.com/). And you can start using it today! Whether you're throwing together a proof of concept for a customer or a prototype for a presentation, the project is able to seamlessly transition into a full-fledged powerful enterprise app, no matter which platforms or sets of devices you are developing for.

### About this series

This post is part of a series on kick-starting your development with Ionic and Generator-M-Ionic, so without any further ado here is the fun-map of our three-act-adventure:

- The rest of this part: Take your first baby steps with Generator-M-Ionic in our island's toddler-playground
  - preparations
  - generating your first app
  - learn about file structure and git integration
- [Part 2](http://tutorials.pluralsight.com/front-end-javascript/advanced-workflows-for-building-rock-solid-ionic-apps-part-2-mountain): Climb to the top of serious-app-development-mountain
  - quality assurance and testing
  - adding Angular components, Sass, Cordova Plugins and bower packages
  - run your app in your browser and on a device using livereload
  - integrate into different ecosystems like [Ionic Platform](http://ionic.io/) using the [Ionic CLI](http://ionicframework.com/docs/cli/)
- [Part 3](http://tutorials.pluralsight.com/front-end-javascript/advanced-workflows-for-building-rock-solid-ionic-apps-part-3-orbit): Lift your app into adventure-orbit
  - ease through backend issues with environments and a CORS proxy
  - learn about powerful build tools like resource sets and build vars
  - some tips regarding continuous integration and delivering your app
  - take a peak beyond future-horizon and see what kind of treasures the future might hold

So pack your shovels, buckets and the rest of your building-equipment and meet me in the playground to set up your first app!

![Generator-M-Ionic technology stack](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/tech_stack.png)

### Get your hands dirty

While I will try to explain everything from the ground up, I do expect you to have some experience in web development and development with Angular and Ionic in general.

So before we dive into the sandbox we'll have to check whether your system is ready for cross-platform HTML5 app development. The first thing you need to install is [Node](https://nodejs.org/en/) and the **Platform SDKs** for the platforms you want to develop for. The latter is only needed if you want to run your app on a real device during development. Have you already built apps using the [Ionic CLI](http://ionicframework.com/docs/cli/)? Then chances are high you already have those installed and don't need to do it again. You haven't? Then follow our [Installation and Prerequisites Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/installation_prerequisites.md).

After you've set up everything properly launch your terminal -if you haven't done so already- and run:
```sh
npm install --global generator-m-ionic bower yo gulp
```
This will install Generator-M-Ionic using the [node package manager](https://www.npmjs.com/) along with the popular web development tools [Bower](http://bower.io/) (manage and install client packages like Angular), [Yeoman](http://yeoman.io/) (runs the generator and scaffolds your app) and [Gulp](http://gulpjs.com/) (runs all post-setup tasks). If you don't know what these are, the [What's in the box Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/intro/whats_in_the_box.md) can help shed some light on the technologies Generator-M-Ionic uses.


### Create your project
Once all of this is done, you probably want to create a new folder to generate your project in and launch the generator:
```sh
# create new directory and change into it
mkdir adventure-island && cd $_
# launch generator
yo m-ionic
```

You'll be asked a series of questions regarding the project you're creating, like this one:

![Questions the generator asks](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/generator_question.png)

If you're not sure what some of these mean, they are explained in the [Questions introduction document](https://github.com/mwaylabs/generator-m-ionic/tree/master/docs/start/questions.md). For the rest of this series I'm going with `tabs` as a starter template. When you've answered all the questions, all the node and bower dependencies will be installed for your project. This might take a while but luckily you'll only have to do this once for each project.

![npm install](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/npm_install.png)

After everything's done run:
```sh
gulp watch
```
Your default browser will open up automatically. If you don't want that, adding the `--no-open` flag will prevent opening your browser or a new window. Now activating your developer tools (`cmd+alt+i` in Chrome on OS X) will let you see your app as below.

<p align="center">
  <img alt="view Generator-M-Ionic app" src="https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/gulp_watch.png" width="330px">
</p>

Congratulations you've built your first app! This might not look like much now, but the generator has done A LOT of work for you already. Let's have a closer look:

### File structure
Your project folder now contains a lot of files that were generated for you. In order to give you a first general orientation, the most important of these files are:

#### Cordova related files and folders
```sh
config.xml  # configuration of your Cordova project
platforms/  # platforms you installed
plugins/    # plugins you installed
www/        # will contain the build of your web app, before Cordova is added
```

#### Gulp tasks and dependencies
```sh
bower.json    # dependencies like Angular & Ionic installed to app/bower_components/
gulpfile.js   # configuration of all the Gulp tasks
gulp/         # more Gulp tasks
package.json  # dependencies like Gulp plugins installed to node_modules/
```

#### Application files
```sh
app/
  └── index.html # single most important file, everything is wired together here
  └── app.js     # include modules YOU create, for now only 'main'
  └── main/      # all other app files, angular files, styles, assets, ...
    └── main.js  # routing, angular module dependencies
    └── ...
```

#### index.html
Everything that makes up your app comes together in this file: Cordova, Ionic, Angular, Angular modules, your own application logic, styles and assets. I'll give some more details here without going into the particulars too much:

When you ran `gulp watch` it did this to your `index.html`:
- inject all bower javascript and css files (Angular, Ionic, ...)
- inject all of your app files (compiled css, angular files, ...)

And whenever you make changes to your application files or if you create new ones, `gulp watch` will update all necessary files and reload your browser. Additionally -besides the necessary HTML and angular bootstrapping we just discussed- the `index.html` contains:
- the link to the `cordova.js` file that Cordova provides upon build
- basic ionic directives used by the routing in your `main.js`

### Git integration
Now that you have created your first app with Generator-M-Ionic and wired it all together with `gulp watch`, it might be a good time to secure your progress! Good thing your project comes with a fine-tuned [Git](https://git-scm.com/) integration already! So simply type the following in your project folder:

```sh
git init    # initialize git
git status  # show all files that git can track
```

You'll notice that we have already talked about most of the files that Git finds. Worry not, we will discover the rest of them step by step throughout this series. However you might also notice that Git ignores all the third party code of Bower, Node and Cordova, namely the `app/bower_components/`, `node_modules/`, `platforms/` and `plugins/` folders. This is done using the `.gitignore` and `.gitattributes` files which got accordingly configured during project setup. Thus Git ignores all files that aren't source files or in any other way specific to your project, keeping your repository nice and small and your commits clutter free. For the curious of you there's a [Git integration Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/git_integration.md) in the documentation with more particulars.

So to create your first commit, all you have to do is:
```sh
git add . # add all relevant files
# create commit
git commit -m "project setup with yo m-ionic and gulp watch"
```
**Side note:** I'm using double quotes on any terminal related strings, because they're safe on all systems. On OS X or Linux, using single quotes is just fine.

### Nice!
You've taken your first steps with [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), learned about file structure, `gulp watch` and Git integration! Take your development to the next level in [part 2](http://tutorials.pluralsight.com/front-end-javascript/advanced-workflows-for-building-rock-solid-ionic-apps-part-2-mountain) of our series covering quality assurance, adding a variety of ingredients to your app and integrating into different ecosystems like [Ionic Platform](http://ionic.io/).

### Get in touch
Feedback, ideas, comments regarding this blog post or any of the features discussed here are very welcome in either the comments section below, at our [Generator-M-Ionic's Github repository](https://github.com/mwaylabs/generator-m-ionic) or the [Generator-M-Ionic Gitter Chat](https://gitter.im/mwaylabs/generator-m-ionic).

### Credits
Author: [Jonathan Grupp](https://github.com/gruppjo)  
Headline illustrations: [Christian Kahl](http://www.art-noir.net/)  
Special thanks to Volker Hahn, Mathias Maier & Tim Lancina

> Originally published July 5, 2016 on the [Ionic Blog](http://blog.ionic.io/advanced-workflows-for-building-rock-solid-ionic-apps-part-1/) in a slightly modified version.
