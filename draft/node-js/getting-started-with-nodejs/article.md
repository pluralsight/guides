In this article, I am going to introduce NodeJS with Node Package Module (NPM), step-by-step basic implementation and explanation.

This article covers the following areas of NodeJS.

* Introduction of NodeJS
* Installation of NodeJS and NPM
* Node Package Module (NPM)
* Package.json
* Basic Example

## Introduction of NodeJS 

[NodeJS](https://nodejs.org/en/) is an open-source, cross-platform runtime environment for developing server-side web application. NodeJS has an event-driven architechture capable of asynchronous I/O.

NodeJS uses an event-driven, non-blocking I/O model that makes it lightweightand efficient.

## Installation of NodeJS and NPM

Installation of NodeJS and NPM is straightforward using installer package available at NodeJS official web site.

#### Installation steps

1. Download the installer from [NodeJS WebSite](https://nodejs.org/en/).
2. Run the installer.
3. Follow the installer steps, agree the license agreement and click the next button.
4. Restart your system/machine.

Now, test NodeJS by printing its version using command

```
> node -v
```

and test npm by printing its version using command

```
> npm -v
```

Simple way to test nodeJS work in your system is to create a javascript file which print a message.

Lets create test.js file 

```
/*test.js file*/
console.log("Node is working");
```

Run the test.js file using Node command `> node test.js` in command prompt.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/640f8199-be76-4848-a093-0f9e2825d8a8.png)

You are done with installation.

## Node Package Module

[NPM](https://www.npmjs.com/) is package module which help javascript developers to load there dependencies in easy and effective manner.

To load dependencies we just have to run a command in dommand prompt

```
> npm install
```

This command is finding a json file named as `package.json` in root directory to install all dependencies defined in the file.

## Package.json

[Package.json](https://docs.npmjs.com/files/package.json) is look like 

```json
{
  "name": "ApplicationName",
  "version": "0.0.1",
  "description": "Application Description",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/npm/npm.git"
  },
  "dependencies": {
    "express": "~3.0.1",
    "sequelize":"latest",
    "q":"latest",
    "tedious":"latest",
    "angular":"latest",
    "angular-ui-router": "~0.2.11",
    "path":"latest",
    "dat-gui":"latest"
  }
}

```

The most important things in your package.json are the name and version fields. Those are actually required, and your package won't install without them. The name and version together form an identifier that is assumed to be completely unique. Changes to the package should come along with changes to the version.

#### Repository 

```json
{
    "repository": {
        "type" : "git",
        "url" : "https://github.com/npm/npm.git"
    }
}
```

Specify the place where your code lives and through this repository developers are reaching to your application who want to contribute. If the git repository is no GitHub, then the `npm docs` command will be able to find you.

#### Script

```
{
 "scripts": {
    "start": "node server.js"
  }
}
```

NPM provide many usefull [Script](https://docs.npmjs.com/misc/scripts) like `npm install`, `npm start`, `npm stop` etc.

Some default script values based on package contents.

```
"start": "node server.js"
```

If there is a server.js file in the root of your package, then npm will default the start command to node server.js.

#### Dependencies

```
{
"dependencies": {
    "express": "~3.0.1",
    "sequelize":"latest",
    "q":"latest",
    "tedious":"latest",
    "angular":"latest",
    "angular-ui-router": "~0.2.11",
    "path":"latest",
    "dat-gui":"latest"
  }
}
```

[Dependencies](https://docs.npmjs.com/files/package.json#dependencies) are specified in a simple object that maps a package name to a version range.
Version Name must be Version exactly.

If you want to install latest version of file you just have to put `latest` on the place of version name.

Tilde(~) is used to tell "Approximately equivalent to version".

## Basic Example

Create server.js file with following code

```
/*server.js*/

const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer(function(req, res) {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, function() {
  console.log('Server running at http://'+ hostname + ':' + port + '/');
});
```
