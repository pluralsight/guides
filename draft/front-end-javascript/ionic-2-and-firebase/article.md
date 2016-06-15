# Ionic 2 and Firebase Part 1

In this guide I will walk you though creating a data service using ionic 2 and Googles Firebase realtime database.

---

## Creating the project

First make sure the ionic cli is installed and up to date. 
```
$ npm install -g ionic@beta 
$ npm install -g cordova 
```
Next lets generate our project. We will be making a simple to do list.

```
$ ionic start todoer blank --v2 --ts
```
Lets take a look at the command. 
* ionic start 
 * creates a new project
* todoer
 * the name of our project
* blank
 * the template we want to use. We are starting a blank project
* --v2
 * Use ionic 2. Since ionic 2 is still in beta start will default to version 1
* --ts
 * Use typescript. Both angular 2 and ionic 2 are writen in typescript, and it appears to be the defacto going forward. So, we will use it too. 

```
$ cd todoer
```

## Setting up firebase
Google recently purchased firebase and has changed the way things are setup so if you already have a firebase account sign in. If not set one up now.

We will create a new project called todoer
## Building the App

Lets go over the key features of this application. 
1. Create todos
2. Mark todos checked or unchecked
3. filter todos by if they are checked or not
4. delete a todo

### Firebase config

### Creating the data service
To act as our mediator between firebase and our app we will be building a data service. For the purpose of this guide we will be housing all of our data in one service. For larger applications you may want to split this out to multiple services. 

First lets create a provider.
```
$ ionic g provider data
```
this will create an @injectable component which we will inject into the root of our app. Why the root you ask? Because we want this service to live for the life time of our application, and not everytime we load a page. Providers that are inject on a per component basis are created at time of injection. So if we want our data to persist we must inject it into the the main app component.

Open up the app.ts file and at the top add:
```
import {Data} from './providers/data/data';
```
Then in the component add Data to the providers;
```
@component({
    providers: [Data]
});
```
#### Firebase reference
Firebase is a realtime nosql database. The way we access data in it is by a query path. For example if our data looks like this.
```
{
  todos: {
    $todoId: {
      title: 'todo1',
      complete: false
    },
    $todoId: {
      title: 'todo2',
      complete: true
    }
  }
}
```
The way we could get access to our list of todos could be ``` firebase.database().ref('/todos');``` If we wanted to get a specific todo we would use ``` firebase.database().ref('/todos/$todoId'); ``` where ``` $todoId ``` is the unique id of the todo. All data in firebase is stored as an object however we can retrive it as an array if we would like. However, we will not be doing that in this tutorial. 

So, lets add firebase to the data provider

```
import {Injectable] from '@angular/core';

@Injectable()
export class Data {
    private _db: any;
    private _todosRef: any;
    
    constructor() {
      this._db = firebase.database().ref('/'); // Get a firebase reference to the root
      this._todosRef = firebase.database().ref('todos'); // Get a firebase reference to the todos
      
    }
```
Now that we have our references to the database how do we get the data?

### Firebase Queries
Firebase has an event base query system which allows you to setup listeners for certian events. For our use case we are going to listen for the ``` child_add``` event. For a complete list of events see firebase's api documentation.

Lets add the query to the constructor of our data service class.
```
import {Injectable] from '@angular/core';

@Injectable()
export class Data {
    private _db: any;
    private _todosRef: any;
    
    constructor() {
      this._db = firebase.database().ref('/'); // Get a firebase reference to the root
      this._todosRef = firebase.database().ref('todos'); // Get a firebase reference to the todos
      this._todosRef.on('child_add', this.handleData, this);
      
    }
    handleData(snap)
    {
        //Do something with the data
    }
```

The line we added setup a listener for when a child is added to the todos path. ``` this.handleData ``` is passed in to handle the data snapshot sent back from firebase. Last we pass ``` this ``` as the context for our handler function.

#### Observers and Observables
For our application to feel as real time as possible we are going to make use of the observer pattern. Observers are built in to angular 2 and currently (at the time of writing) are using RxJS. However, that will be replaced with angulars own implimentation. 

The concept for the pattern is pretty simple. An oberver can subscribe and "observe" an observable. When data is pushed to the observable the observer recieves it until the observable ends. Basically it is a publish subscribe pattern. This is a far over simplification but you get the idea.

In our Data class lets create a ReplaySubject. What is a ReplaySubject? A Subject is both an Observer and an Observerable. Normally an observable will only send the last event to a new subscriber a ReplaySubject however, will send every message to a new subscriber in the order they were sent. This way we will not miss any old todos.

```
import {Injectable] from '@angular/core';
import {ReplaySubject} from 'rxjs/ReplaySubject';

@Injectable()
export class Data {
    private _todos$: any;
    private _db: any;
    private _todosRef: any;
    
    constructor() {
      this._db = firebase.database().ref('/');
      this._todosRef = firebase.database().ref('todos');
       this._todosRef.on('child_add', this.handleData, this);
      this._todos$ = new ReplaySubject();
    }
    get todos()
    {
        return this._todo$;
    }
    
    handleData(snap)
    {
        //Do something with the data
    }
}
```


Lets examine our data. we have a todo:
```
{
    title: 'My todo',
    complete: false,
}
```

Every time a todo is created we want our application to update
### Creating a todo
### Listing the todos
### Editing a todo

## Part 2 -> login
## Part 3 -> proximi
## Part 4 -> publish


