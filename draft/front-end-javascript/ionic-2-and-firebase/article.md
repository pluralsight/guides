
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

## Building the App

Lets go over the key features of this application. 
1. Create todos
2. Mark todos checked or unchecked
3. filter todos by if they are checked or not
4. delete a todo


## Setting up firebase
Google recently purchased firebase and has changed the way things are setup so if you already have a firebase account sign in. If not set one up now.

Once you have your account setup a new project called todoer. After your project is generated click on "add firebase to your web app." this will provide you will all of your api information to connect to firebase. It should look something like this: 
```
<script src="https://www.gstatic.com/firebasejs/3.1.0/firebase.js"></script>
<script>
  // Initialize Firebase
  var config = {
    apiKey: "AIzaSyBBiOO44HzgIdsnzWG8JCic3MmEgsdfJ",
    authDomain: "todoer.firebaseapp.com",
    databaseURL: "https://todoer.firebaseio.com",
    storageBucket: "",
  };
  firebase.initializeApp(config);
</script>
```
Copy that and save it for later.

Now go to the database and select rules. Firebase by default requires you to be authenticated. We are going to change that so anyone can access our app and add authentication later. This way we can get our core functionality done without having to type a username and password everytime we run. Remember this will be changed in a later part.

Change your rules to be this and publish them: 
```
{
  "rules": {
    ".read": true,
    ".write": true
  }
}

```
Select data and add the key `"todos"` to the tree.


### Firebase config

open up your ```www/index.html``` file and paste the you config information from firebase before your other script tags and after the ```<ion-app></ion-app>``` tag.

```
<!DOCTYPE html>
<html lang="en" dir="ltr">

<head>
  <title>Ionic</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">

  <link ios-href="build/css/app.ios.css" rel="stylesheet">
  <link md-href="build/css/app.md.css" rel="stylesheet">
  <link wp-href="build/css/app.wp.css" rel="stylesheet">
</head>

<body>
  <ion-app></ion-app>
  <script src="https://www.gstatic.com/firebasejs/3.1.0/firebase.js"></script>
  <script>
    // Initialize Firebase
    var config = {
      apiKey: "AIzaSyBBiOO44HzgIdsnzWG8JCic3MmEgsdfJ",
      authDomain: "todoer.firebaseapp.com",
      databaseURL: "https://todoer.firebaseio.com",
      storageBucket: "",
    };
    firebase.initializeApp(config);
  </script>
  <!-- cordova.js required for cordova apps -->
  <script src="cordova.js"></script>
  <!-- Polyfill needed for platforms without Promise and Collection support -->
  <script src="build/js/es6-shim.min.js"></script>
  <!-- Zone.js and Reflect-metadata  -->
  <script src="build/js/Reflect.js"></script>
  <script src="build/js/zone.js"></script>
  <!-- the bundle which is built from the app's source code -->
  <script src="build/js/app.bundle.js"></script>
</body>

</html>

```
That's it firebase is ready to go.

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
The way we could get access to our list of todos could be ``` firebase.database().ref('/todos');``` If we wanted to get a specific todo we would use ``` firebase.database().ref('/todos/$todoId'); ``` where ``` $todoId ``` is the unique id of the todo. All data in firebase is stored as an object we can retrive it as an array if we would like. However, we will not be doing that in this tutorial. 

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
Firebase has an event base query system which allows you to setup listeners for certian events. For our use case we are going to listen for the ``` child_added``` event. For a complete list of events see firebase's api documentation.

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
      this._todosRef.on('child_added', this.handleData, this);
      
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
      this._todosRef.on('child_added', this.handleData, this);
      this._todos$ = new ReplaySubject();
    }
    get todos()
    {
        return this._todos$;
    }
    
    handleData(snap)
    {
        try {
            // Tell our observer we have new data
            this._todos$.next(snap.val());
        }
        catch (error) {
            console.log('catching', error);
        }
    }
}
```
In our handle data function we are going to get a snapshot of the data in our database. By calling ```.val()``` function we can get the value associated with that snapshot and we will pass that to our observers. ```this._todos$.next(data)``` sends the new data to everyone setup to listen for it. 


### Creating a todo
Lets examine our data. we have a todo:
```
{
    title: 'My todo',
    complete: false,
}
```

Every time a todo is created we want our application to update Firebase with the new todo. So lets add a function to our data provider that saves a todo.
In the data.ts file

```
save(todo)
{
    this._todosRef.push(todo);
}
```
the push function will add a new todo to Firebase, generate a key for it, and return that key through the ```.key``` property.

Now generate a new page using the ionic cli.
```$ ionic g page new-todo```

You should have three new files new-todo.html, new-todo.ts, and new-todo.scss. Open the new-todo.ts file first and lets add our data service.
```
import {Component} from '@angular/core';
import {NavController} from 'ionic-angular';
import {Data} from '../../providers/data/data';

/*
  Generated class for the NewTodoPage page.

  See http://ionicframework.com/docs/v2/components/#navigation for more info on
  Ionic pages and navigation.
*/
@Component({
  templateUrl: 'build/pages/new-todo/new-todo.html',
})
export class NewTodoPage {
  constructor(public nav: NavController, public _data: Data) {}
}
```
At the top we are going to import our data service, then in the constructor we are to create a reference to it. 

Now our todo has a defined structure to it and we know what types our data members are. So lets make a class object for our todo. Before your NewTodoPage add:
```
class Todo {
  public title: string;
  public completed: boolean;
  constructor() {
    this.title = '';
    this.completed = false;
  }
}
```
Then add a member variable to the NewTodoPage class that has a type of `Todo`.
```
export class NewTodoPage {
  todo: Todo;
  constructor(public nav: NavController, public _data: Data) {
    this.todo = new Todo();
  }
}
```
We define a new todo each time the constructor is called to make sure `this.todo` is not undefined. 

Next lets create our save method. In the NewTodoPage class add:
``` 
save() { 
    var key = this._data.save(this.todo); 
    if(key)
    {
        console.log('saved');
    }
}
```


Thats it for now in our new todo page class. Lets look at the html next.

```
<ion-navbar *navbar>
  <ion-title>new-todo</ion-title>
</ion-navbar>

<ion-content padding class="new-todo">
  
</ion-content>
```

pretty empty so lets our input elements to it. We are going to use an ```ion-list``` and put our input elements in the list items.

in our ```<ion-content>``` tags add:
```
 <ion-list>
    <ion-item>
      <ion-label floating>Title:</ion-label>
      <ion-input  [(ngModel)]="todo.title" type="text"></ion-input>
    </ion-item>
    <ion-item>
      <ion-label floating>Complete</ion-label>
      <ion-checkbox   [(ngModel)]="todo.complete"></ion-checkbox>
    </ion-item>
  </ion-list>
  <button clear (click)="save()">Save</button>
```

So here we see some new Angular 2 things. First the syntax for ``ng-model`` has changed to ``[(ngModel)]``. This new syntax is an expression of its binding model. `{{}}` and `[]` are one-way binding from the data source to the view, `()` is one-way binding from the view to the data source, and the combination `[()]` is two way binding. So our `title` and `complete` are bound to both the data source and the view. Now the ``(click)`` which is a 'statement'. Statements respond to events raised by the element, directive, or component that they are bound to. In our chase that event is a click raised by the button element. Since it is wrapped in `()` we know that it is only going to send data one way; from our veiw to our component. 

For a far more in depth look at this check out Angular 2's docs.
"Ok I made a todo, but I didn't see anything happen?" You are right lets add some user feed back for saving a todo.

Ionic2 has a lot of nice built in components to choose from like Toast. That will provide some nice feed back. To add a Toast to our new todo page first we will need to import it. 
```
import {NavController, Toast} from 'ionic-angular';
```

In our save method lets add:
```
var key = this._Data.save(todo); 
    if(key)
    {
      let toast = Toast.create({
        message: 'Todo Saved',
        duration: 500
      });
    
      toast.onDismiss(() => {
        this.nav.pop()
        console.log('Dismissed toast');
      });
    
      this.nav.present(toast);
    }

```
If we receive a key from firebase create a new toast and present it. Once dissmised we will remove the page from the stack.Ionic2's routing works like a stack you push on new views and you pop off old ones.

Home (push)=> New Todo (push)=> otherView <= user sees, but the other views are still there. When the user hits the back button the top state pops off of the stack. 
Home (push)=> New Todo <=(pop) <= user sees. 

Once we make our todos we will want to see them in our list view right? Lets do that now.

### Listing the todos

Instead of generating a new page for the list we are going to use the already generated home page. This way as soon as a user accesses the app they see their todos. 

Like with the new page we are going to add the data service to the constructor. We are, also, going to add a subsciption to our todo observable so we will get updates as soon as they happen.

```
import {Component} from "@angular/core";
import {NavController} from 'ionic-angular';
import {Data} from '../../providers/data/data';
import {NewTodoPage} from '../new-todo/new-todo';

@Component({
  templateUrl: 'build/pages/home/home.html'
})
export class HomePage {
  public todos: any[] = [];
  constructor(private _navController: NavController, private _data: Data) {
    let that = this;
    this._data.todos.subscribe((data) => {that.todos.push(data);}, (err) => {console.error(err);});
  }
  newTodo()
  {
    this._navController.push(NewTodoPage);
  }
}
```
Notice we, also, imported our new-todo page and added a newTodo method. This is what we will call when changing to make a new todo. In that method we are calling our nav controllers push method. 

Now that we have our data setup lets render it. In our html we will have this:
```
<ion-navbar *navbar>
  <ion-title>
    Ionic 2 Todo App
  </ion-title>
</ion-navbar>

<ion-content class="home">
  <ion-list *ngIf="todos">
    <ion-item *ngFor="let todo of todos">
      <ion-label>{{todo.title}}</ion-label>
      <ion-checkbox [(ngModel)]="todo.complete"></ion-checkbox>
    </ion-item>
  </ion-list>
  <p *ngIf="!todos"> No todos </p>
  <button fab fab-bottom fab-right (click)="newTodo()"> New </button>
</ion-content>
```
Some more new concepts here angular2's new ```*ngIf``` as compared to Angular 1's ```ng-hide``` and ```ng-show``` to check if we have any todos to show. If we do we are looping through our todos using the ``*ngFor`` replacement for ``ng-repeat`` then we display our todo using the ngModel syntax we saw earlier. If we do not have any todos we are doing a simple paragraph to tell the user that. Then finally we are calling the method we made earlier to take us to create a todo. 

You might be asking what is with that `*` before these directives? That is syntax that angular 2 uses to help with reading and writing directives that modifiy html.


That concludes part 1 in part 2 we will edit a todo and add user authentication.




