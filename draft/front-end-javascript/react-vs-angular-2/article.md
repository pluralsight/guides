This guide is aimed to provide a critical overview of two popular front-end web development tools - [React](https://facebook.github.io/react/)  and [Angular 2](https://angular.io/). The tools will be evaluated based on their approaches to building applications, involvement and adoption by the community, performance and ability to integrate with different platforms. 

The goal of this guide is to delineate all the differences and similarities between the two tools in order facilitate decision-making when it comes to choosing which one appropriate for a given project specification.


# Structuring code
Angular 2 is written in TypeScript, a superset language of JavaScript developed by Microsoft. The language introduced types, new data structures and more object-oriented features, making the code easier to read and maintain compared to other alternative.It uses [TSLint](http://palantir.github.io/tslint/) to check the code for issues. Using TypeScript makes setting up an Angular 2 application somehow tediuos, as it introduces extra overhead for configuration. 

React is relying on JSX, a XML-esque syntax extension for rendering JavaScript and HTML. Syntax and strucutre-wise, JSX looks more familiar to JavaScript and apperas to blend into HTML more easily. It uses [ESLint](http://eslint.org/) to track for isses in the code.

To get a clearer comparison between React and Angular 2, we are going to compare how a basic TODO application is built (example apps built by [Mark Volkmann (mvolkmann)](https://github.com/mvolkmann/react-examples).

 Both applications use [Webpack](https://webpack.github.io/) for development and deployment.


##### Angular 2

*package.json*

```json                                  
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
##### React

*package.json*
```json
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

##### Angular 2

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

##### React

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


Because both setups use Webpack to handle all the assets, the index.html file is identical.

Applicaiton structure-wise, both React and Angular 2 need two files - <code>TodoList</code> that represents the list of tasks and <code>Todo</code> that represents a single task.

I'll analyize the <code>TodoList</code>  component (*todolist.component.ts* for Angular 2 and *todo-list.js* for React, respectively) piece-by-piece, from top to bottom:

#### **Importing** 

##### Angular 2


```ts
import 'angular2/bundles/angular2-polyfills';
import {Component} from 'angular2/core';
import {bootstrap} from 'angular2/platform/browser';
import {TodoCmp, ITodo} from "./todoCmp";
```
##### React


```js
import React from 'react';
import ReactDOM from 'react-dom';
import Todo from './todo';
```

#### **Initializing**

##### Angular 2

```ts
// Used for type of state property in TodoListCmp class.
// Both properties are optional, but preferred as they make 
// the code more maintainable.
interface IState {
  todos?: ITodo[];
  todoText?: string;
}
 
// This syntax is called a Decorator and is similar
// to Annotations in other languages.  Decorators are
// under consideration to be included in ES2016.
@Component({
  // other components used here
  directives: [TodoCmp],
  // TodoListCmp's HTML element selector
  selector: 'todo-list',
  template: // see value to left of
            // React render method below
})

export class TodoListCmp {
  private static lastId: number = 0;
  private state: IState;

  constructor() {
    this.state = {
      todos: [
        TodoListCmp.createTodo('learn Angular', true),
        TodoListCmp.createTodo('build an Angular app')
      ]
    };
  }
```

##### React


```js
let lastId = 0; // no static class properties in ES6

class TodoList extends React.Component {
  constructor() {
    super(); // must call before accessing "this"
    this.state = {
      todos: [
        TodoList.createTodo('learn React', true),
        TodoList.createTodo('build a React app')
      ]
    };
  }
```

#### **Creating an item**

##### Angular 2

```ts
static createTodo(
    text: string, done: boolean = false): ITodo {
    return {id: ++TodoListCmp.lastId, text, done};
}

onAddTodo(): void {
    const newTodo: ITodo =
      TodoListCmp.createTodo(this.state.todoText);
    this.state.todoText = '';
    this.state.todos = this.state.todos.concat(newTodo);
}
```

##### React
```js
static createTodo(text, done = false) {
    return {id: ++TodoList.lastId, text, done};
}

onAddTodo() {
    const newTodo =
      TodoList.createTodo(this.state.todoText);
    this.setState({
      todoText: '',
      todos: this.state.todos.concat(newTodo)
    });
}
```
#### **Detecting item states**

##### Angular 2

```ts
get uncompletedCount(): number {
    return this.state.todos.reduce(
      (count: number, todo: ITodo) =>
        todo.done ? count : count + 1, 0);
}
```
##### React 
```js
get uncompletedCount() {
    return this.state.todos.reduce(
      (count, todo) =>
        todo.done ? count : count + 1,
      0);
}

```

####  **Reacting to changes**

##### Angular 2
```ts
onArchiveCompleted(): void {
    this.state.todos =
      this.state.todos.filter((t: ITodo) => !t.done);
}

onChange(newText: string): void {
    this.state.todoText = newText;
}

onDeleteTodo(todoId: number): void {
    this.state.todos = this.state.todos.filter(
      (t: ITodo) => t.id !== todoId);
}

onToggleDone(todo: ITodo): void {
    const id: number = todo.id;
    this.state.todos = this.state.todos.map(
      (t: ITodo) => t.id === id ?
        {id, text: todo.text, done: !todo.done} : t);
}
```

##### React
```js
onArchiveCompleted() {
    this.setState({
      todos: this.state.todos.filter(t => !t.done)
    });
}

onChange(name, event) {
    this.setState({[name]: event.target.value});
}

onDeleteTodo(todoId) {
    this.setState({
      todos: this.state.todos.filter(
        t => t.id !== todoId)
    });
}

onToggleDone(todo) {
    const id = todo.id;
    const todos = this.state.todos.map(t =>
      t.id === id ?
        {id, text: todo.text, done: !todo.done} :
        t);
    this.setState({todos});
}
```
#### **Bootstrapping**

##### Angular 2
```
} // end of TodoListCmp class

// Each Angular app needs a bootstrap call
// to explictly specify the root component.
// In larger apps, bootstrapping is usually in
// a separate file with more configuration.
bootstrap(TodoListCmp);
```
#####  React
```
} // end of TodoList class

// This renders a TodoList component
// inside a specified DOM element.
// If TodoList was used in more than one place,
// this would be moved to a different JavaScript file.
ReactDOM.render(
  <TodoList/>,
  document.getElementById('container'));
```
#### **Template syntax**

##### Angular 2
```

  // This is the value of the @Component template
  // property above.  It was moved here for
  // easy comparison with the React render method.
  `<div>
    <h2>To Do List</h2>
    <div>
      {{uncompletedCount}} of
      {{state.todos.length}} remaining
      <!-- Clicking this button invokes the
           component method onArchiveCompleted. -->
      <button (click)="onArchiveCompleted()">
        Archive Completed
      </button>
    </div>
    <br/>
    <form>
      <!-- [ngModel] tells where to get the value. -->
      <!-- (input) tells what to do on value change. -->
      <input type="text" size="30" autoFocus
        placeholder="enter new todo here"
        [ng-model]="state.todoText"
        (input)="onChange($event.target.value)"/>
      <button [disabled]="!state.todoText"
        (click)="onAddTodo()">
        Add
      </button>
    </form>
    <ul>
      <!--
        This uses a for loop to generate TodoCmps.
        #todo defines a variable to use within the loop.
        [todo] sets a property on each TodoCmp.
        (onDeleteTodo) and (onToggleDone) are outputs
        from the TodoCmp, and they call functions
        on TodoListCmp with emitted values.
        deleteTodo receives a type of number.
        toggleDone receives a type of ITodo.
      -->
      <todo *ngFor="let todo of state.todos" [todo]="todo"
        (on-delete-todo)="onDeleteTodo($event)"
        (on-toggle-done)="onToggleDone($event)"></todo>
    </ul>
  </div>
```

##### React
```  
render() {
    // Can assign part of the generated UI
    // to a variable and refer to it later.
    const todos = this.state.todos.map(todo =>
      <Todo key={todo.id} todo={todo}
        onDeleteTodo=
          {this.onDeleteTodo.bind(this, todo.id)}
        onToggleDone=
          {this.onToggleDone.bind(this, todo)}/>);

    return (
      <div>
        <h2>To Do List</h2>
        <div>
          {this.uncompletedCount} of
          {this.state.todos.length} remaining
          <button
            onClick={() => this.onArchiveCompleted()}>
            Archive Completed
          </button>
        </div>
        <br/>
        <form>
          <input type="text" size="30" autoFocus
            placeholder="enter new todo here"
            value={this.state.todoText}
            onChange=
              {e => this.onChange('todoText', e)}/>
          <button disabled={!this.state.todoText}
            onClick={() => this.onAddTodo()}>
            Add
          </button>
        </form>
        <ul className="unstyled">{todos}</ul>
      </div>
    );
 }

```

The component code for Angular 2 is more verbose than React's . TypeScript largely contributes to the this. <code> :void </code> , <code> :ITodo </code> and other types are often seen in the  Style-wise, varaibles, function parameters and function themselves must have a type in TypeScript. React does not have any of that, making the code more convenient for developers who prefer traditional JavaScript. 

Angular 2 uses <code>@Component</code> decorators to specify the different constructs (services, directives, pipes) that it contains and to define its own properties. It also lets the developer specify a <code>selector</code> for the component and put the code for the template using the <code> template </code> property or the <code> templateUrl</code> if a separate file is used. The same applies for styles - there is an option to input styles directly using the <code> styles </code> property or to include an array of files using <code> styleUrls </code>. React does not have an analog for a component decorator and requires the template of the component to be included in the component file itself, thus limiting the options for customization and making the information about a component harder to interpret.

Angular 2 templates come with a specific convention for different elements:
- ```*``` denotes a directive that manipulates the DOM. For example, ```*ngFor``` iterates over an array of values and creates a new element for each. ```*ngIf``` is used to denote whether an element can be seen or not.
- ```[]``` denotes a property binding, indicating that the element receives a value from the component. It is used mostly to pass data to a child component to the parent.
- ```()``` indicates an event binding (such as ```(click)```, which triggers a function in the components.
- ```[()]``` indicates two-way binding, meaning that any changes to the data that are passed will be propagated.

React templates also come with their peculiarities. JSX blurs the line by html and JavaScript, providing its own markup. Even though it seems that the template is written in HTML, the markup is actually XML that is later parsed into HTML.
Because of JSX, standard html property words are camel-cased or have diffeerent names - <code>onclick</code> in JSX is <code>onClick</code>. <code>for</code> (used in labels) becomes <code>htmlFor</code> as <code>for</code> is a reserved word in JavaScript for a loop.


//todo finish this up
# Data flow and Application architecture

## Angular 2 

![angular 2 data flow](https://raw.githubusercontent.com/pluralsight/guides/master/images/f5c9fff0-9d0c-4e5c-8cc2-aa646d5c2c8b.png)

Angular 2 is using the standard bi-directional data flow model. Once an action is made, the data goes up to the service layer, then it goes back to the view. This way of manipulating data has been widely criticised because the state of the data can be changed both ways, resulting in inconsistencies.

Recently, Angular 2 has started adopting the [Redux](http://redux.js.org/docs/introduction/ThreePrinciples.html) pattern for data flow by utilizing [rxjs](https://splintercode.github.io/is-angular-2-ready/)'s observables to maintain the state of the data and using pure functions to manipulate it. [Attemps have been made](http://blog.rangle.io/getting-started-with-redux-and-angular-2/) to build unidirectional data flow in Angular 2 applications, but the adoption of this technique is still low.

## React

![react data flow](https://raw.githubusercontent.com/pluralsight/guides/master/images/032be950-9bac-4a65-9bba-0a33089aad37.png)

React uses [flux](https://facebook.github.io/flux/docs/overview.html) as a way of constructing its applications. Each action is sent to a dispatcher, which sends the new information to stores. Stores then update (mutate) the information  and send it down to the views (controller-views), which listen for changes.

React also uses Redux, and since Redux  was first introduced for React and was on simplifying Flux it is more widely adopted in the React community. Another reason for Redux's wide  adoption in the React community is that it shares many with characteristics Flux (unidirectional data flow, dependable mutations, etc).



# Development flow

Angular CLI , etc

React- more freedom
   

# Adoption
[Angular 2](https://github.com/angular/angular) just shy of 16K stars on Github, currently in RC5, check [here](https://splintercode.github.io/is-angular-2-ready/) for release.

[React]


# Performance

# Cross-platform integration

NativeScript vs React Native 
