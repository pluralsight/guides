# What is Webpack?

Webpack is a module bundler. It takes the code you write and bundles it. But Webpack can also transpile, combine, and minify your code. It allows for code splitting, in which the client can load blocks of code on demand rather than having to send one huge file to the client. Webpack is also compatible with React.js.

Webpack requires that the code be divided into modules. You can use any module system (AMD, CommonJS or ES6 Modules). This guide will use ES6 modules, but feel free to adapt the solutions to the module system of your choice.

### Why Webpack?

Using a module bundler like Webpack solves a lot of problems. Loading individual javascript files in an application will trigger multiple calls to the server. Processing each of those requests and sending the resources to the client takes time. The bigger the application, the longer the time, Thus, having lots of files and requests is simply bad UX. 

By combining the files you drastically reduce the number of requests to the server. Furthermore, when the application is built across multiple files, the size of those files also comes into play. Minifying the combined file through a production grade minifier reduces the size of the files you request.

Another problem of loading multiple files is that of dependencies and load order. Resolving them is a huge cognitive load. This is where Webpack comes in. Since Webpack needs the code to be split into modules, its easier to resolve and understand dependencies.

Webpack is not the only solution out there. Task runners like Grunt or Gulp can also get the job done. Since both of these tools are built around a plugin ecosystem, you can pick and choose plugins based on your requirements.

### Installing Webpack

Webpack is available as a Node package. To install Webpack:

```
$ npm install -g webpack
```

This command will install Webpack globally and is available from the command line through the ```webpack``` command. It's important to note that Webpack works only with Node and not Bower. 

#### CLI (Command Line Interface)

Now that we have access to the ```webpack``` command, we can create two files: app.js and index.html.

In app.js:

```
console.log("Loading...");

document.write("Well, hello there Webpack!!!!");
```

In index.html:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Getting started with Webpack: Part 1</title>
</head>
<body>
    <script src="bundle.js"></script>
</body>
</html>

```

Now in the command line, type the command:

```
webpack ./app.js bundle.js
```

The first argument in the command is the input file, or the file on which Webpack will operate. The second argument is the output file. So when this command is executed you should see a new file called ```bundle.js``` in the directory.

Now, it would be cumbersome if you have to do this everytime you change something in your code. To fix that, Webpack can `watch` for changes in your file and automatically create the output file. To do this, type the following command:

```
webpack ./app.js bundle.js --watch
```

Adding the ```--watch``` will start watching the app.js file for changes. Whenever webpack detects a change, it will bundle the code into the output file. Webpack will also do a production build of your code. 

```
webpack ./app.js bundle.js -p
```

Now open ```bundle.js``` and voila! Your code is minified and production ready.

# webpack.config.js

Obviously, typing out the command in the terminal is not practical. Moreover, when you have multiple operations you want to perform - compile LESS/Sass files to CSS, CoffeeScript to Javascript and transpile ES6 to ES5, it's better to configure webpack to take care of the operations for us. This is where the ```webpack.config.js``` file comes in. 

Create a file in your project root folder and name it ```webpack.config.js```. Next, install ```babel``` and ```css-loader```.

```
npm install style-loader css-loader babel-loader babel-core babel-preset-es2015 --save-dev
```

Now in the ```webpack.config.js``` file:

```
module.exports = {
    entry: ['./app.js'],
    output: {
        path: __dirname,
        filename: 'bundle.js'
    },
    module: {
        loaders: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                include: path.join(__dirname, 'src'),
                loaders: ['babel'],
                query: {
                    presets: ['es2015']
                }
            },
            {
                test: /\.css$/,
                loaders: ['style-loader!css-loader']
            }
        ]
    }
}
```

##### entry

The ```entry``` property lets you define the top level file(s) that we want to include in the bundle.

##### output

The ```output``` object let's you define the output configuration. The ```path``` property defines the path to the root of the project. The ```filename``` property defines the name of the output file. 

##### module loaders

The ```loaders``` array contains the configurations for each loader module. 

```test``` - Regular expression mapping to the file extension of the files that you want to apply the module to.

```exclude``` - Regular expression mapping to the files/folders that you want to exclude

```include``` - Files/folder that you want to include

```loaders``` - Name(s) of loaders that you want to apply

```presets``` - Lets webpack know that what kind of code we're writing, in our case ES2015

Now that we have our config file ready, let's install ```webpack-dev-server```.

```
npm install webpack-dev-server -g
```

Now run:

```
webpack-dev-server
```

Open your favorite browser and go to ```http://localhost:8080/webpack-dev-server/``` and see the app running. If you make a change in your source files, webpack automatically runs your code through the loaders, bundles them up, and refreshes the page, thanks to our `--watch` command. 

I hope this provided you with a glimpse into the power of using Webpack. Feel free to leave your comments and feedback in the discussion section below. 

____

Part 2 will cover plugins, debugging and Webpack developer tools. Stay tuned!