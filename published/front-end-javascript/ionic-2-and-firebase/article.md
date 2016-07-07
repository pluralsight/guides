#### In this guide I will walk you though creating a data service using Ionic 2 and Google's Firebase realtime database.

---

## Creating the project

First make sure that the Ionic CLI (Command Line Utility) is installed and up to date. 
```
$ npm install -g ionic@beta 
$ npm install -g cordova 
```
Next, let's generate our project. We will be making a simple to-do list.

```
$ ionic start todoer blank --v2 --ts
```
Lets take a closer look at the commands we just used: 
* `ionic start` 
 * creates a new Ionic project
* `todoer`
 * names our project
* `blank`
 * identifies the template we want to use. In this case, we are starting a blank project.
* `--v2`
 * Identifies that we are using ionic 2. Since ionic 2 is still in beta, start defaults to version 1 unless another version is specified
* `--ts`
 * Use TypeScript. Both Angular 2 and Ionic 2 are writen in TypeScript, and it appears to be the default going forward. So, we will use it too. 

```
$ cd todoer
```

## Building the App

Lets go over the key features of this application. 
1. Create tasks (or todos)
2. Mark tasks as checked or unchecked
3. Filter checked/unchecked tasks
4. Delete a task


## Setting up firebase
Sign into or create your Google Firebase account. Firebase is Google's realtime database.

Once you have your account ready, create a new project called "todoer". After this, click on "Add Firebase to your web app." This will provide you with all of your API information to connect to Firebase. It should look something like this: 
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

Now go to the database and select **rules**. Firebase by default requires you to be authenticated. We are going to change that so anyone can access our app, and we add authentication later. This way we can get our core functionality done without having to type a username and password everytime we run. Remember, this will be changed later.

Publish your adjusted rules using the following command: 
```
{
  "rules": {
    ".read": true,
    ".write": true
  }
}

```
Select data and add the key `"todos"` and the value of `''` to the tree.


### Firebase config

Open up your ```www/index.html``` file, and paste the config information from Firebase after your ```<ion-app></ion-app>``` tag and before your other script tags.

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
  <!-- Start Add -->
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
  <!-- End Add -->
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
Firebase is now ready to go.

### Creating the data service
To act as our mediator between firebase and our app we will be building a data service. For the purpose of this guide we will be housing all of our data in one service. For larger applications you may want to split this out to multiple services. 

First lets create a provider.
```
$ ionic g provider data
```
This will create an @injectable component which we will inject into the root of our app. Why the root you ask? We want this service to live for the life time of our application, and not everytime we load a page. Providers that are inject on a per component basis are created at time of injection. So, if we want our data to persist we must inject it into the the main app component.

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
The way we could get access to our list of todos could be ``` firebase.database().ref('/todos');``` If we wanted to get a specific todo we would use ``` firebase.database().ref('/todos/$todoId'); ``` where ``` $todoId ``` is the unique id of the todo. All data in firebase is stored as an object.

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
Now that we have our references to the database, how do we get the data?

### Firebase Queries
Firebase has an event base query system which allows you to setup listeners for certain events. For our use case, we are going to listen for the ``` child_added``` event. For a complete list of events, see Firebase's API documentation.

Let's add the query to the constructor of our data service class.
```
import {Injectable] from '@angular/core';

@Injectable()
export class Data {
    private _db: any;
    private _todosRef: any;
    
    constructor() {
      this._db = firebase.database().ref('/'); // Get a firebase reference to the root
      this._todosRef = firebase.database().ref('todos'); // Get a firebase reference to the todos
      this._todosRef.on('child_added', this.handleData, this); // ***ADD THIS LINE***
      
    }
    handleData(snap)
    {
        //Do something with the data
    }
}
```

The line we added to our constructor set up a listener for when a child is added to the `todos` path. ``` this.handleData ``` is passed in to handle the data snapshot sent back from Firebase. Last we pass ``` this ``` as the context for our handler function.

#### Observers and Observables

For our application to feel as close to realtime as possible, we are going to make use of the observer pattern. Observers are built into Angular 2 and (at the time of writing) use RxJS (ReactiveX library for JavaScript). However, that library will be replaced with Angular's own library implementation. 

The concept for the pattern is pretty simple. An observer can subscribe to and "observe" an observable. When data is pushed to the observable, the observer receives it until the observable ends. To oversimplify, the observe-observable relationship is a publish-subscribe pattern.

Now that we understand observers, let's move on with our app. In your Data class, create a ReplaySubject. What is a ReplaySubject? A Subject is both an Observer and an Observable. Normally an observable will only send the **last event** to a new subscriber. A ReplaySubject however, will send **every message** to a new subscriber in the order they were sent. Slightly less efficient, but this way we will not miss any old todos.

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
In our `handleData()` function, we are going to get a snapshot (hence `snap`) of the data in our database. By calling the ```.val()``` function we can get the value associated with that snapshot and we will pass that to our observers. ```this._todos$.next(data)``` sends the new data to everyone setup to listen for it. 


### Creating a task

Every time a todo is created we want our application to update Firebase. So let's add a function to our data provider that saves a todo.

In your `data.ts` file, add the following method:

```
save(todo)
{
   return this._todosRef.push(todo).key;
}
```
The push function will add a new todo to Firebase, generate a key for it, and return that key through the ```.key``` property.

Now, generate a new page using this Ionic shell command.
```$ ionic g page new-todo```

You should have three new files:  `new-todo.ts`, `new-todo.html`, and `new-todo.scss`. Open the `new-todo.ts` file first, and add our data service.
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
At the top we are going to import our data service. Then in the constructor we are to create a reference to the data service. 

Now our todo has a definite structure to it, and we know what our data types will be. So, let's make a class object for our todo. Before your NewTodoPage class add:
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

Next comes the `save()` method. In the NewTodoPage class add:
``` 
save() { 
    var key = this._data.save(this.todo); 
    if(key)
    {
        console.log('saved');
    }
}
```
We check the key to make sure our push was successful. Firebase will return null if it did **not** create the todo.

That is it for now in our NewTodoPage class. Let's look at the html next.

```
<ion-navbar *navbar>
  <ion-title>new-todo</ion-title>
</ion-navbar>

<ion-content padding class="new-todo">
  
</ion-content>
```

Pretty empty. Let's add our input elements to it. We are going to use an ```ion-list``` and put our input elements into the list items.

in our ```<ion-content>``` tags add:
```
 <ion-list>
    <ion-item>
      <ion-label floating>Title:</ion-label>
      <ion-input [(ngModel)]="todo.title" type="text"></ion-input>
    </ion-item>
    <ion-item>
      <ion-label floating>Complete</ion-label>
      <ion-checkbox [(ngModel)]="todo.complete"></ion-checkbox>
    </ion-item>
  </ion-list>
  <button clear (click)="save()">Save</button>
```

So, here we see some new Angular 2 things. The syntax for ``ng-model`` has changed to ``[(ngModel)]``. This new syntax is an expression of its binding model. `{{}}` and `[]` are one-way binding from the data source to the view and `()` is one-way binding from the view to the data source. The combination `[()]` is **two-way binding**. So our `title` and `complete` are bound to both the data source and the view, hence the realtime nature of the application. 

Now let's discuss ``(click)``, a kind of statement. Statements respond to events raised by the element, directive, or component that they are bound to. In our case, that event is a click raised by the button element. Since it is wrapped in `()` we know that it is only going to send data one way; from our veiw to our component. 

For a far more in depth look at statements, check out Angular 2's docs.

Now if our user makes a todo they have no way of knowing if it was successful. So lets add some user feedback.

Ionic 2 has a lot of nice built-in components to choose from, including the Toast component. Toasts are a great way to provide feedback. To add a Toast to our new todo page, we will first need to import Toasts from Ionic 2. 
```
import {NavController, Toast} from 'ionic-angular';
```

In our `save()` method, lets add:
```
var key = this._data.save(todo); 
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
If we receive a key from Firebase, we'll create a new toast and present it. Once dissmised we will remove the page from the stack. 

Ionic 2's routing works like a stack; you push on new views and you pop off old ones.

**Home (push) => New Todo (push) => otherView <= user sees, but the other views are still there. **

When the user hits the back button, the top state pops off of the stack. 

**Home (push) => New Todo (pop) <= user sees.**

Once we make our todos, we will want to see them in our list-view right? Lets do that now.

### Listing the todos

Instead of generating a new page for the list, we are going to use the existing home page. This way, as soon as a user accesses the app, they see their todos. 

Like with the new page we are going to add the data service to the constructor. We are also going to add a subsciption to our todo observable so we will get updates as soon as they happen.

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
Notice we also imported our new-todo page, and added a newTodo method. This is what we will call when changing to the newTodoPage. In that method, we are calling our nav controller's push method. 

Now that we have our data set up, let's render it. In our html we will have this:
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
Some more new concepts here: Angular2's new ```*ngIf``` is used, as opposed to Angular 1's ```ng-hide``` and ```ng-show```, in order to check if we have any todos to show. 

If we do have todos to show, we will loop through our todos using the ``*ngFor`` replacement for ``ng-repeat``. Then we display our todo using the two-way ngModel syntax we saw earlier. 

If we do not have any todos, we are doing a simple paragraph to tell the user that we do not have todos to show. Then we call the `newTodo()` method we created earlier to take us to create a todo. 

You might be wondering about the `*` before these directives. That is syntax that Angular 2 uses to help with reading and writing directives that modifiy html.

Now run `ionic serve` from your command line to start our application and check it out.
____

##### That concludes part 1! In part 2, we will edit a todo and add user authentication.
____
Thank you for reading this tutorial. Please upvote if you found it useful and/or interesting. Also, feel free to make comments and give feedback in the discussion section below.




