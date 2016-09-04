This guide is aimed to provide a critical overview of two popular front-end web development tools - [React](https://facebook.github.io/react/)  and [Angular 2](https://angular.io/). The tools will be evaluated based on their approaches to building applications, involvement and adoption by the community, performance and ability to integrate with different platforms.

# Key differences
### Structuring code
Angular 2 is written in TypeScript, a superset language of JavaScript developed by Microsoft. The language introduced types, new data structures and more object-oriented features, making the code easier to read and maintain compared to other alternative.It uses [TSLint](http://palantir.github.io/tslint/) to check the code for issues.

Using TypeScript makes setting up an Angular 2 application somehow tediuos, as it introduces extra overhead for configuration. 

React is relying on JSX, a XML-esque syntax extension for rendering JavaScript and HTML. Syntax and strucutre-wise, JSX looks more familiar to JavaScript and apperas to blend into HTML more easily. It uses [ESLint](http://eslint.org/) to track for isses in the code.

To get a clearer comparison between React and Angular 2, we are going to compare how a basic TODO application is built (example apps built by [Mark Volkmann (mvolkmann)](https://github.com/mvolkmann/react-examples).

 Both applications use [Webpack](https://webpack.github.io/) for development and deployment.


**Angular 2** 

*package.json*

````json                                  
{                                     
  "name": "ng2-todo",
  "version": "1.0.0",
  "description": "Todo app in Angular",
  "scripts": {
    "start": "webpack-dev-server
      --content-base src --inline --port 8081"
  },
  "devDependencies": {
    "ts-loader": "^0.7.2",
    "tslint": "^3.0.0-dev.1",
    "typescript": "^1.7.3",
    "webpack": "^1.12.9",
    "webpack-dev-server": "^1.14.0"
  },
  "dependencies": {
    "angular2": "2.0.0-beta.0",
    "es6-promise": "3.0.2",
    "es6-shim": "0.33.3",
    "reflect-metadata": "0.1.2",
    "rxjs": "5.0.0-beta.0",
    "zone.js": "0.5.10"
  }
}| 
```
** React **

*package.json*
```
{
  "name": "react-todo",
  "version": "1.0.0",
  "description": "Todo app in React",
  "scripts": {
    "start": "webpack-dev-server
      --content-base . --inline"
  },
  "devDependencies": {
    "babel-core": "^6.1.2",
    "babel-loader": "^6.0.1",
    "babel-preset-es2015": "^6.1.18",
    "babel-preset-react": "^6.1.2",
    "eslint": "^1.6.0",
    "eslint-loader": "^1.0.0",
    "eslint-plugin-react": "^3.5.1",
    "webpack": "^1.12.9",
    "webpack-dev-server": "^1.14.0"
  },
  "dependencies": {
    "react": "^0.14.3",
    "react-dom": "^0.14.3"
  }
}
```
Both applicaitons need a similar amount of packages for development. React has more devDependencies for using React-specific linting and parsing.

**Angular 2**

*index.html*

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Angular Todo App</title>
    <link rel="stylesheet" href="todo.css">
  </head>
  <body>
    <todo-list>Loading...</todo-list>
    <script src="bundle.js"></script>
  </body>
</html>

```

**React**

*index.html*

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Angular Todo App</title>
  </head>
  <body>
    <todo-list>Loading...</todo-list>
    <script src="bundle.js"></script>
  </body>
</html>
```


Both main html files are slim, since both applicaitons use Webpack for figuring out asset structure and injecting code.

Both the React and Angular 2 applications have two files - <code>**TodoList**</code> that represents the list of tasks and <code>Todo</code> that represents a single task.

## Tooling and development flow
Angular CLI , etc

React- more freedom
   

# Adoption


# Performance

# Cross-platform integration
