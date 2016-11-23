This guide will provide a critical overview of two popular front-end web development tools - [React](https://facebook.github.io/react/)  and [Angular 2](https://angular.io/). The tools will be evaluated based on their approaches to building application structure, adoption by the community, performance, and ability to integrate with different platforms. 

The goal of this guide is to highlight core differences and understand the similarities between the two tools, thus facilitating the decision-making process when it comes to choosing which tool is appropriate for a given project specification.


# Development flow

### Angular 2
Angular 2 is very close to being a fully-fledged framework - it comes with many building blocks out of the box that cover most of the common scenarios in developing a web application. There is also a clear separation of the roles of the different elements:


- *Services* - Injectable elements that are used to consume data from an API or share state between multiple components.
- *Components* - Building blocks of the user interface that consume services. Can be nested inside each other through structural directive selectors.
- *Directives* - Divided into structural and attribute directives. Structural directives (ex. <code>*ngFor</code>)  manipulate the DOM (Document Object Model) while attribute directives are part of elements and control their style and state.
- *Pipes* - Used to format how data is displayed in the view layer.
- *Modules* - Exportable blocks of the application that isolate components, directives, pipes, services and routes together.
- 
### React

**React is far less rigid than Angular 2**. It is a library that provides the most basic tools for building a web applications - a HTTP service and Components.  There is no built-in router or anything that sets a particular convention. It is mostly up to the developer's choice what kind of packages he/she is going to use to shape the application.



# Code structure

### Angular 2

Angular 2 is written in TypeScript, a superset language of JavaScript developed by Microsoft. The language introduces types, new data structures and more object-oriented features, making the code easier to read and maintain compared to vanilla JavaScript. Initially, TypeScript's syntax was intended to be reminiscent of C#. However, recent veriouns of TypeScript feature the arrow (`=>`) notation and other ES6-specific syntax, making TS resemble more ES6. 

Using TypeScript makes setting up an Angular 2 application somehow tedious, as it introduces extra overhead for configuration. 


### React

React is relying on JSX (Java Serialization to XML), an XML-esque syntax extension for rendering JavaScript and HTML. In terms of syntax and structure, JSX looks like JavaScript and appears to blend into HTML more easily. It allows the mixing  of variables and JavaScript data structures within the HTMl markup.



**To get a clearer comparison between React and Angular 2's code strucures, we are going to compare how a basic TODO application is built** (example apps built by [Mark Volkmann (mvolkmann)](https://github.com/mvolkmann/react-examples).

 Both applications use [Webpack](https://webpack.github.io/) for development and deployment.



In terms of application structure, both React and Angular 2 need two files - <code>TodoList</code> which represents the list of tasks and <code>Todo</code> which represents a single task.

Let's analyze the <code>TodoList</code> component (*todolist.component.ts* for Angular 2 and *todo-list.js* for React, respectively) piece-by-piece, from top to bottom:

#### **Importing** 

##### Angular 2


```ts
import {Component} from '@angular/core';
import {bootstrap} from from '@angular/platform-browser-dynamic';;
import {TodoCmp, ITodo} from "./todoCmp";
```
##### React


```js
import React from 'react';
import ReactDOM from 'react-dom';
import Todo from './todo';
```

#### **Initializing a component**

I've spotted the key differences between the tools using comments. 

##### Angular 2

```js
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
  selector: 'todo-list',
  template:`  ` /*This is where the markup for the template can be put.
   Alternatively, you can reference a html file using templateUrl */
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

As seen above, the creation process is quite similar for both tools, but React's is slightly shorter and appears to need less developer logic.

#### **Detecting item states**

##### Angular 2

```js
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
        todo.done ? count : count + 1, 0);
}

```

####  **Reacting to changes**

##### Angular 2
```js
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

```js
//In Angular 2, chunks of the
//application logic that are responsible for a certail feature
//are encapsualted in a module
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { TodoListCmp }   from './todoList.component';
@NgModule({
  imports:      [ BrowserModule ],
  declarations: [ TodoListCmp ],
  bootstrap:    [ TodoListCmp ]
})
export class AppModule { }
//An Angular module can import functionalities from other modules and export some of its building blocks to other modules.

```

```js
// src/main.ts

import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app.module';

//The module is bootstrapped, making the bootstrapped component the root //of the application if no routes are defined.

const platform = platformBrowserDynamic();
platform.bootstrapModule(AppModule);

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

The component code for Angular 2 is generally more verbose than React's. The only case we've covered where React requires more code is with bootstrapping. This disparity can be attributed to TypeScript. <code> :void </code> , <code> :ITodo </code> and other types are often seen in the TypeScript. Angular 2 style usually dictates that variables, function parameters, and function themselves must have a type in TypeScript. React does not have any of that, making the code more convenient for developers who prefer traditional, more succint JavaScript. 

Angular 2 uses <code>@Component</code> decorators to specify the different constructs (services, directives, pipes) that it contains and to define its own properties. It also lets the developer specify a <code>selector</code> for the component and put the code for the template using the <code> template </code> property or the <code> templateUrl</code> if a separate file is used. The same applies for styles - there is an option to input styles directly using the <code> styles </code> property or to include an array of files using <code> styleUrls </code>. React does not have an analog for a component decorator and requires the template of the component to be included in the component file itself, thus limiting the options for customization and making the information about a component harder to interpret.

Angular 2 templates come with a specific convention for different elements:
- ```*``` denotes a directive that manipulates the DOM. For example, ```*ngFor``` iterates over an array of values and creates a new element for each. ```*ngIf``` is used to denote whether an element can be seen or not.
- ```[]``` denotes a property binding, indicating that the element receives a value from the component. It is used mostly to pass data to a child component to the parent.
- ```()``` indicates an event binding (such as ```(click)```, which triggers a function in the components.
- ```[()]``` indicates two-way binding, meaning that any changes to the data that are passed will be propagated.

React templates also come with their peculiarities. JSX blurs the line by HTML and JavaScript, providing its own markup. Even though it seems that the template is written in HTML, the markup is actually XML that is later parsed into HTML.
Because of JSX, standard HTML property words are camel-cased or have diffeerent names - <code>onclick</code> in JSX is <code>onClick</code>. <code>for</code> (used in labels) becomes <code>htmlFor</code> because <code>for</code> is a reserved word in JavaScript (meaning for-loop).


#### **Nesting components**

Next, let's look into how the component for a single ToDo item looks like in Angular 2 and in React:


##### Angular 2

```js
import {Component, Input, Output, EventEmitter}
  from 'angular2/angular2';

export interface ITodo {
  id: number;
  text: string;
  done: boolean;
}

@Component({
  selector: 'todo',
  template: `
    <li>
      <input type="checkbox"
        [checked]="todo.done"
        (change)="toggleDone()"/>
      <span [ng-class]="'done-' + todo.done">
        {{todo.text}}
      </span>
      <button (click)="deleteTodo()">Delete</button>
    </li>`
})
export class TodoCmp {
  // @Input allows this component to receive
  // initialization values from the containing component.
  @Input() todo: ITodo;
  // @Output allows this component to
  // publish values to the containing component.
  @Output() onDeleteTodo:
    EventEmitter<number> = new EventEmitter<number>();
  @Output() onToggleDone:
    EventEmitter<ITodo> = new EventEmitter<ITodo>();

  deleteTodo(): void {
    this.onDeleteTodo.next(this.todo.id);
  }

  toggleDone(): void {
    this.onToggleDone.next(this.todo);
  }
}
```

##### React

```js
import React from 'react';

// There are three ways to define React components.
// This is the stateless function component form
// which only receives data through "props".
// A props object is passed to this function
// and destructured.
const Todo = ({onDeleteTodo, onToggleDone, todo}) =>
  <li>
    <input type="checkbox"
      checked={todo.done}
      onChange={onToggleDone}/>
    <span className={'done-' + todo.done}>
      {todo.text}
    </span>
    <button onClick={onDeleteTodo}>Delete</button>
  </li>;

// Optional validation of props object.
const PropTypes = React.PropTypes;
Todo.propTypes = {
  todo: PropTypes.object.isRequired,
  onDeleteTodo: PropTypes.func.isRequired,
  onToggleDone: PropTypes.func.isRequired
};

export default Todo;

```

Because it is the main building block of a React application, the component offers greater flexibility in the way it can be structured. Stateless components are built with one-directional data flow in mind, leaving the majority of the logic to stay outside of the component itself. Angular 2 also offers similar functionality, but it does it with more complexity.


# Application Architecture

### Angular 2 

![angular 2 data flow](https://raw.githubusercontent.com/pluralsight/guides/master/images/f5c9fff0-9d0c-4e5c-8cc2-aa646d5c2c8b.png)
*Bi-directional data flow typical for MVC patterns*

Angular 2 is using the standard bi-directional data flow model. Once an action is made, the data goes up to the service layer, then it goes back to the view. This way of manipulating data has been widely criticised because the state of the data can be changed both ways, resulting in inconsistencies.



### React

![react data flow](https://raw.githubusercontent.com/pluralsight/guides/master/images/032be950-9bac-4a65-9bba-0a33089aad37.png)
*Flux data flow*

React uses [Flux](https://facebook.github.io/flux/docs/overview.html) to construct its applications. Each action is sent to a dispatcher, which sends the new information to stores. The stores then update (mutate) the information and send it down to the views (controller-views), which listen for changes.


### Redux

![Redux](https://raw.githubusercontent.com/pluralsight/guides/master/images/f685aa1c-99d2-44aa-b705-99267c16bd4b.com-optimize)
*Redux data flow*

Redux is the cool kid on the block. Like Flux, it is based on one-directional data flow, but it manages the data in a different way.

An application that uses Redux has a global state that is shared among the whole application. When an action is made, it is sent to a dispatcher, which serves to make a HTTP call to the API. Then, the changes are sent to reducers - pure functions that specialize in managing a part of the global application state. For example, if your application has users, projects and items as models, each of the models will be part of the global state and there will be a set of reducers that take care of managing their state. Once the state is 'mutated' by the reducers, the new information is sent down to the view.

In this way,  state of the application is kept in one place, providing for an easier state management.

Redux was first introduced for React as an alternative to simplify Flux. Because of that, Redux is more widely adopted in the React community.

Recently, Angular 2 has started adopting the [Redux](http://redux.js.org/docs/introduction/ThreePrinciples.html) pattern for data flow by utilizing [rxjs](https://splintercode.github.io/is-angular-2-ready/)'s observables to maintain the state of the data and using pure functions to manipulate it. [Attemps have been made](http://blog.rangle.io/getting-started-with-redux-and-angular-2/) to build unidirectional data flow in Angular 2 applications, but the adoption of this technique is still low.




# Performance

 Performance is another critical point of comparison. It shows how Angular 2 and React handle under heavy load and gives predictions for future problems as the applications become bulkier and more difficult to optimize.
 
 The following tests have been developed by [Stefan Krauss (krausest)](https://github.com/krausest). The tests are carried out with **[Selenium](http://docs.seleniumhq.org/docs/03_webdriver.jsp)** to emulate user actions and record the results (in miliseconds)


![benchmarks](https://raw.githubusercontent.com/pluralsight/guides/master/images/08a35dbd-3654-42b5-a423-bd8ba9c2406e.001)



Both Angular 2 and React are seem on par, with Angular 2 having a slight advantage on all DOM manipulations except for creating new elements.

In terms of memory management, React outperforms Angular 2 by using less memory, especially after page load. This is mainly due to the fact that React itself is smaller-sized than Angular 2.


# Cross-platform integration

The cross-platform development world has made a huge leaps since the massive adoption of mobile devices. Now, WebView-based tools for building cross-platform mobile applications are obsolete. 

There are two tools that allow building of cross-platform applications with native capabilities for Angular 2 and React, respectively - [NativeScript](https://www.nativescript.org/) and [React Native](https://facebook.github.io/react-native/)

NativeScript, built by [Telerik (now Progress)](http://www.telerik.com/), is intended to enable native mobile applications to be built with Angular 2. Even though it has only [8k](https://github.com/NativeScript/NativeScript) stars on Github, NativeScript is mature and comes with great documentation and numerous extensions. It preaches a "write once, run everywhere" methodology of developing cross-platform applications. Even though this is an efficient approach, it does not enable devleopers to use many platform-specific UI features.

On the other hand, React Native, developed by Facebook, gives React native capabilities. Compared to NativeScript, React Native is still young but quite popular and maturing quickly, boasting over [37k](https://github.com/facebook/react-native) stars on GitHub. React Native uses a "learn once, write anywhere" approach to developing cross-platform applications. Instead of attempting to generalize different platforms like NativeScript does, Rect Native embraces their differences, offering different ways to construct UI for the different platforms. Recently, the Angular 2 team has started adopting React Native, [making a renderer for it](https://github.com/angular/react-native-renderer).

# Adoption
[Angular 2](https://github.com/angular/angular) is just over 18k stars on Github. There are [~15 000](https://github.com/search?l=TypeScript&p=3&q=angular+2&type=Repositories&utf8=%E2%9C%93) repositories on Github that contain "Angular 2" or "ng2" and are written in TypeScript

[React](https://github.com/facebook/react), on the other hand, is over 53K stars on Github. There are [95, 500](https://github.com/search?l=JavaScript&p=1&q=react&ref=searchresults&type=Repositories&utf8=%E2%9C%93) repositories containing the word "react" in them, roughly over nine times more than what Angular 2 has.

Still, one of the big reasons for the large number of repositories is that React comes with just a few built-in functionalities and relies on its community to provide it with the needed tooling to develop full-scale applications.


# Conclusion 

Both Angular 2 and React have large communities and boast corporate backing. Even though they have different philosophies to how an application development must be approached, there is a place for both of them in the software development world.
