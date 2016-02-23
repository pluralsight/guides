# Installation and first steps

## Install node.js
Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine for developing server-side applications. It also has a built in package ecosystem called npm (node package manager), which allows you to install a variety of packages and plugins (such as Ember CLI) to increase your development efficiency.

### nvm
While node has a great installer, I highly recommend installing node via nvm (Node Version Manager), which allows you to install and run multiple versions of node alongside one another.

## Install Ember CLI
This part is simple. Run the command `npm install -g ember-cli`. Before we move on, though, let's look at what this command does. `npm` invokes the node package manager executable. This command takes a variety of subcommands such as:
- `install` - install a package
- `help` - lookup help on subcommands or the npm command itself
- `update` - update a package
- `ls` - list installed packages
We invoke the `install` subcommand with a "flag" (a letter preceded by a dash such as `-g` or `-l`). The `-g` flag tells npm to install this package globally. This means that the package will be accessible from anywhere that has access to your node modules, instead of just the particular project your are working on. Finally `ember-cli` is the name of the package we install. When installed it exposes an `ember` executable that is used to manage Ember CLI-based projects.

## Generate a project
Generating a project in Ember CLI is simple. First, make sure you're in the directory where you'd like to store the project (for example, `~/Projects` or `~/dev`). Once you're in an appropriate directory, we'll use the `ember new` command to start a new project by passing it a project name. We'll be developing a simple to-do list application, so we can run `ember new todos` to create our project folder.