# Installation and first steps

## Install node.js
Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine for developing server-side applications. It also has a built in package ecosystem called npm (node package manager), which allows you to install a variety of packages and plugins (such as Ember CLI) to increase your development efficiency.

### nvm
While node has a great installer, I highly recommend installing node via nvm (Node Version Manager), which allows you to install and run multiple versions of node alongside one another. Why would you want to do such a thing? Let's say you're working as a consultant and constantly moving back and forth between many different projects. Each of these projects would likely be created at different points in time, and, therefore, reliant on different versions of tools. Because node.js is constantly in development, you may be running projects under different versions of node that aren't necessarily compatible. nvm gives us a simple interface to manage the different versions of node on our system, and easily switch between them as we jump between different projects.

#### Installing on OS X
To install nvm on OS X, we can use the fantastic homebrew package manager. Once you have it installed, simply run `brew install nvm`. Once it's been installed we have a few housekeeping items to address to make sure that it's properly configured. First, create a working directory for the tool by calling `mkdir ~/.nvm`. This tells our system to "make a directory" (`mkdir`) in our "home" folder (`~`), called `.nvm`. The file is preceeded by a `.` to make it a "hidden" folder, ensuring that it won't show up in your typical filesystem browsing activities (because you shouldn't typically be directly modifying its contents).

Next, we need to add some environment configuration to our shell's configuration (`.zshrc` or `.bashrc`). At the time of this writing, nvm requires the following configuration:
```
export NVM_DIR=~/.nvm
. $(brew --prefix nvm)/nvm.sh
```
Finally, load the configuration by "sourcing" your `rc` file - (`source ~/.zshrc` or `source ~/.bashrc`). You can run `nvm help` to ensure everything is working at this point.

## Install Ember CLI
With node.js and nvm successfully setup, simply run the command `npm install -g ember-cli`. Before we move on, though, let's look at what this command does. `npm` invokes the node package manager executable. This command takes a variety of subcommands such as:

- `install` - install a package
- `help` - lookup help on subcommands or the npm command itself
- `update` - update a package
- `ls` - list installed packages

We invoke the `install` subcommand with a "flag" (a letter preceded by a dash such as `-g` or `-l`). The `-g` flag tells npm to install this package globally. This means that the package will be accessible from anywhere that has access to your node modules, instead of just the particular project your are working on. Finally `ember-cli` is the name of the package we install. When installed it exposes an `ember` executable that is used to manage Ember CLI-based projects.

## Generate a project
Generating a project in Ember CLI is simple. First, make sure you're in the directory where you'd like to store the project (for example, `~/Projects` or `~/dev`). Once you're in an appropriate directory, we'll use the `ember new` command to start a new project by passing it a project name. We'll be developing a simple to-do list application, so we can run `ember new todos` to create our project folder. After the command completes, run `cd todos` (or assuming you named it `todos`) to move to your new project directory.

## Viewing the project
Ember CLI gives us a fully functional ember application straight "out of the box." Let's jump right in by running `ember serve` in our new projects directory. The project should be viewable at http://localhost:4200. While our project doesn't look like much at this point, we've actually got a full webserver running with live-reloading capabilities and much, much more with very little effort. Furthermore, the Ember CLI tool has a full compliment of generators, a solid "addon" ecosystem, and production-ready build tool to assist in deploying our apps.

## The Ember Inspector
Now that we've got our "app" up and running, it's time to take a quick detour and look at another powerful Ember development tool - the [Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en). The Ember Inspector gives you a full suite of debugging and exploration capabilities right in the browser.

# The application route and index route
One of Ember's core differentiators is it's focus on routing as a first-class citizen. Unlike other JavaScript frameworks that view routing as an afterthought or something to be handled by a plugin, Ember has full fledged support for dynamic routing all handled from the client-side. This came from a mindset of JavaScript apps that "don't break the back button" and work in compliment with the features of modern browsers, rather than competing against them.

Out of the box, every Ember application has a root-level route known as the "application" route. Let's take a look at what this means, why it's necessary, and how it affects the rest of our applicaiton.

# Our first route

Now that we have a decent understanding of the routes that Ember builds by default, let's take a crack at building our own route.