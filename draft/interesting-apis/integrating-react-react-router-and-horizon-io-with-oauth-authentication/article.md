# Introduction
In this tutorial, we're going to develop an application that shows how to integrate React, React Router, and Horizon.io with OAuth authentication from beginning to end.

The application is simple. It just stores and presents messages in real-time:

![Demo App](https://raw.githubusercontent.com/pluralsight/guides/master/images/06bc5c51-4a09-4c4e-9aa6-61c1fd1e0440.gif)

You can find the code on [Github](https://github.com/eh3rrera/react-horizon).

# Requirements
[Horizon.io](http://horizon.io/) is a real-time backend for Javascript apps built on top of [RethinkDB](https://rethinkdb.com/). If you don't know this NoSQL database, [here's a tutorial that shows how it works](http://tutorials.pluralsight.com/nosql-databases/a-practical-introduction-to-rethinkdb).

So first, you'll need to install RethinkDB. There are packages for all major operative systems, [here are the instructions](http://rethinkdb.com/docs/install/).

Horizon is a [Node.js](https://nodejs.org/) application, so you'll also need version 4.4 or higher of Node.js and npm installed. You can download an installer for your platform [here](https://nodejs.org/en/download/).

Then, install Horizon by executing:
```
npm install -g horizon
```

About [React](https://facebook.github.io/react/), you don't need to be a guru to follow this tutorial, but you'll need to have some basic knowledge about components and how this library works.

Let's get started.

# Creating an Horizon app
First, execute the following command:
```
hz init react-horizon
```

This will create a Horizon application in the directory `react-horizon` with following structure:
```
.hz
    config.toml
dist	
	index.html
src	
```

`.hz/config.toml` is the [TOML](https://github.com/toml-lang/toml) configuration file for the Horizon server.

`dist` is the directory where the public and static files will be stored.

For authentication, Horizon requires us to work with `https`, so we need to configure a certificate. Luckily, Horizon comes with a tool to create a self-signed certificate (you just need to have OpenSSL installed), so `cd` into this directory and execute the command:
```
cd react-horizon
hz create-cert
```

This will create `horizon-cert.pem` and `horizon-key.pem`. By default, Horizon will look for these files in the root directory of the application when starting a secure server. If you want to change the name or the location of these files, uncomment and change the following section of the `.hz/config.toml` file:
```
###############################################################################
# HTTPS Options
# 'secure' will disable HTTPS and use HTTP instead when set to 'false'
# 'key_file' and 'cert_file' are required for serving HTTPS
#------------------------------------------------------------------------------
# secure = true
# key_file = "horizon-key.pem"
# cert_file = "horizon-cert.pem"
```

Now, the command to start the server in a dev environment is:
```
hz serve --dev
```

The `--dev` flag will set the following flags:
- `--start-rethinkdb` that will start a RethinkDB server automatically
- `--secure no` that will start an insecure server (with no SSL)
- `--permissions no` that will disable the permissions system
- `----auto-create-collection` and `--auto-create-index` that will create tables and indexes if they don't exist
- `--server-static ./dist` that will configure `dist` as the directory from which the static content will be served

However, since we are going to use HTTPS, we need to redefine the `secure` option, so start the server with the following command:
```
hz serve --dev --secure yes
```

The output of this command should be similar to the following:
```
App available at https://127.0.0.1:8181
RethinkDB
   ├── Admin interface: http://localhost:33361
   └── Drivers can connect to port 41477
Starting Horizon...
🌄 Horizon ready for connections
```

If you go to https://localhost:8181 you should see this (after accepting the warning of the self-signed certificate):

![Horizon initial app](https://raw.githubusercontent.com/pluralsight/guides/master/images/decc6b53-04af-41a9-9360-0034b5f82d56.gif)

Moreover, a RethinkDB server will be started automatically and a `rethinkdb-data` directory will be created. When you go to http://localhost:33361 (or whatever address Horizon gives you in the console), to the *Tables* section you should see the following:

![RethinkDB dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/550bcc76-6bea-4c52-b251-df9bb2be1235.png)

As you can see, Horizon has created two databases with the name of the project. `react_horizon` will store the data used by the application into collections, while `react_horizon_internal` stores metadata about these collections, users, and groups.

# Setting up React
We're going to use ECMAScript 2015 so let's set up [Babel](https://babeljs.io/) to transform this syntax to one most browser can understand:
```
echo '{ "presets": ["react", "es2015", "stage-0"] }' > .babelrc
```

Babel 6.x does not ship with any transformations enabled, so you need to explicitly tell it what transformations to run by using a preset.

I think the first two are very self-descriptive. The stage-x presets are changes to the language that haven’t been approved to be part of a release of Javascript.

The [TC39](https://github.com/tc39) categorizes proposals into 4 stages:

- stage-0 - Strawman: just an idea, possible Babel plugin. 
- stage-1 - Proposal: this is worth working on.
- stage-2 - Draft: initial spec.
- stage-3 - Candidate: complete spec and initial browser implementations.
- stage-4 - Finished: will be added to the next yearly release.

`stage-0` includes all plugins from presets of all levels. `stage-1` includes all plugins from presets 1 to 4 and so on.

To execute Babel and bundle our scripts with their dependencies, we'll use [Webpack](https://webpack.github.io) and npm. So let's installed (globally) with:
```
npm install -g webpack
```

And add a `package.json` configuration file with:
```
npm init
```

Or if you want to accept all the defaults:
```
npm init -y
```

Next, install the Babel dependencies and presets (and a polyfill to emulate a full ES2015 environment) with:
```
npm install --save-dev babel-core babel-loader
npm install --save-dev babel-preset-es2015 babel-preset-react babel-preset-stage-0 babel-polyfill
```

Do the same with Webpack and React:
```
npm install --save-dev webpack
npm install --save react react-dom react-router
```

Now, create a `webpack.config.js` file with the following content:
```
var path = require('path');

module.exports = {
    entry: ["./src/app.js"],

    output: {
        filename: "dist/js/bundle.js",
        sourceMapFilename: "dist/js/bundle.map"
    },

    devtool: '#source-map',
    
    module: {
        loaders: [
            {
                loader: 'babel',
                exclude: /node_modules/
            }
        ]
    }
}
```

This way, Webpack will create a `dist/js/bundle.js` file with all the JavaScript code of the application (from the script `/src/app.js`).

Finally, let's add to `package.json` the following `start` script to start our application:
```
{
  ...
  "scripts": {
    "start": "webpack && hz serve --dev --secure yes"
  },
  ...
}
```

# Creating the app with React and React Router
Let's start by defining the HTML file that will contain our React application, `dist/index.html`:
```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>React.js + Horizon.io</title>
    <link rel=stylesheet href=/css/style.css />
  </head>
  <body>
    <h1>React.js + Horizon.io</h2>
   <div id="root"></div>
   <script src="/js/bundle.js"></script>
  </body>
</html>
```

And the CSS to style it, `dist/css/style.css`:
```css
body {
	background:#300637;
	padding:4em;
	text-align:center;
	font-family: 'Arial, Helvetica, sans-serif';
	color:#c0c0c0;
}

h1 {
	font-weight:100;
}

.menu {
    margin-top: 3em;
    margin-bottom: 5em;
}

.menu-option {
	color:#999;
	background:rgba(0, 0, 0, 0.5);
	padding:1em 5em;
	font-size:0.8em;
	text-decoration:none;
	letter-spacing:2px;
	text-transform:uppercase;
}

.menu-option:hover {
	background:rgba(0, 0, 0, 0.4);
	color:#fff;
}

.menu-option.active {
	border:none;
	background:rgba(0, 0, 0, 0.4);
	background:#fff;
	padding:1.5em 5em;
	color:#000;
	font-size:0.9em;
}

.login-btn {
	border:none;
	color:#999;
    background:rgba(0, 0, 0, 0.5);
	padding:1.5em 5em;
	font-size:0.9em;
	text-transform:uppercase;
    cursor: pointer;
}

.message-input {
    display: block;
    margin: 0.5em;
    padding: 0.8em;
    width: 90%;
    border: none;
    font-family: sans-serif;
    font-size: .9em;
}

.message-container {
	margin: 2em auto;
	padding: 0.8em;
	width: 80%;
}

.message-btn {
	padding: .75rem;
	margin: .5em auto 1.5em;
	width: 6em;
	text-align: center;
    cursor: pointer;
}

.message-row {
    padding: .75em;
    margin: .75em;
    box-shadow:0 0 1px 1px #999;
}
```

`src` will store the JavaScript code of our application, so let's start by defining the starting point, `app.js`:
```javascript
import React from 'react';
import ReactDOM from 'react-dom';

import Router from './router';

ReactDOM.render(Router, document.getElementById('root'));
```

This way, `router.js` will contain all the routes of the application:
```javascript
import React from 'react';
import { Router, Route, browserHistory, IndexRoute } from 'react-router';

// Layout
import MainLayout from './components/main-layout';

// Pages
import Home from './components/home';
import MessageContainer from './components/message-container';
import Login from './components/login';

export default (
  <Router history={browserHistory}>
    <Route path="/" component={MainLayout}>
      <IndexRoute component={Home} />
      <Route path="messages" component={MessageContainer} />
      <Route path="login" component={Login} />
    </Route>
  </Router>
);
```

We'll have three nested routes (so `MainLayout` can serve as a template):
- `/` is our home
- `/login` is our login page
- `messages` is the page with the message functionality

Now let's create the components that represent these pages in the `components` directory.

`components/main-layout.js` contains the general layout of the app:
```javascript
import React, { Component } from 'react';
import { Link } from 'react-router';
import Menu from './menu';

export default class MainLayout extends Component {

    render() {
        return (
	        <div className="app">
                <div>
                    <Menu/>
                </div>
                <div>
                    {this.props.children}
                </div>
            </div>
        );
    }
}
```

The menu is delegated to the `components/menu.js` component:
```javascript
import React, { Component } from 'react';
import { Link, IndexLink } from 'react-router';

export default class Menu extends Component {

    render() {
        var menu = <div className={'menu'}>
                    <IndexLink to="/" className={'menu-option'} activeClassName="active">Home</IndexLink>
                    <Link to="/messages" className={'menu-option'} activeClassName="active">Messages</Link>
					<Link to="/login" className={'menu-option'} activeClassName="active">Login</Link>
                </div>;

        return (
	        menu
        );
    }
}
```

An `<IndexLink>` is like a `<Link>`, but it's only active when the current route is exactly the linked route (otherwise, since all these are nested routes, `/` and `/messages` will be marked as active at the same time).

For now, `components/login.js` will just contain a button to login with Github:
```javascript
import React, { Component } from 'react';

export default class Login extends Component {

    render() {
        return (
	        <div>
                <button className={'login-btn'}>Login with Github</button>
            </div>
        );
    }
}
```

`components/home.js` will contain:
```javascript
import React, { Component } from 'react';

export default class Home extends Component {

    render() {
        return (
	        <h1>Home</h1>
        );
    }
}
```

While `components/message-container.js` will contain:
```javascript
import React, { Component } from 'react';

export default class MessageContainer extends Component {

    render() {
        return (
	        <h1>Message Container</h1>
        );
    }
}
```

If you run the application at this point with:
```
npm start
```

It should look like this:

![React Checkpoint](https://raw.githubusercontent.com/pluralsight/guides/master/images/24244a55-7ecf-48ec-8004-7a6ce00e3e3a.gif)

# Adding real-time functionality with Horizon