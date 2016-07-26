#### In this guide I will walk you though editing a todo, and adding user authentication using Ionic 2 and Google's Firebase realtime database.
##### Note: This is part 2 in the series.

---

## Where we left off
At the end of part 1 we had our application able to connect to firebase, list todos, and create new todos. So our next step is going to be editing a todo. 

##### Note: Since writing part 1 the Angularfire 2 library has reached beta and is a good choice for any angular firebase application. However, since we have already started down the road of writing everything ourselves we will continue.

### The Dataservice
To start off we will add a new method to our data service called `update` it will take a todo object "`_todo`" and the id "`_id`" of the todo we want to update. In that method we will call our firebase refernce to the todos list, find the todo to update, update it, and return the promise.

Now in firebase there are two functions we could call to do this update; set and update. Both will update the object at `_id` the difference is that `set` is a destructive action. Meaning that it will completely rewrite the object at `_id`. Where as update will merge the old data and the new data only updating the fields that changed. Example:

```
_id_1: {
  title: 'test',
  complete: false
}

this._todosRef.child(_id_1).set({title: 'set test'}); // => _id_1: { title: 'set test' }
this._todosRef.child(_id_1).update({title: 'update test'}); // => _id_1: { title: 'update test', complete: false }
```
So we will add this to our data service class in data.ts.

```
update(_id, _todo)
{
    return this._todosRef.child(_id).update(_todo)
}
```
Next we will need access to the key firebase creates for each todo, but the way we are currently storing our todos we throw that data away. Lets change the `handleData` method to address that issue.

```
handleData(snap) 
{ 
  try { 
    // Tell our observer we have new data 
    // this._todos$.next(snap.val()); // <- Change this
    this._todos$.next({key: snap.key, val: snap.val()}); // <- To this
  } 
  catch (error) 
  { 
    console.log('catching', error); 
  }
}

```

Why don't we just pass the snap instead of making a new object. There is a lot of info in the snap that we really don't need right now, and creating a new object slims it down to just the values we are planning on using. 

With that update we are going to need to change our list page. So in home.html change the ngFor to use `todo.val` instead of just `todo`.

```
<ion-label>{{todo.val.title}}</ion-label>
<ion-checkbox [(ngModel)]="todo.val.completed"> {{todo.val.title}} </ion-checkbox>
```
 
 We need to update the todo object everytime the checkbox is clicked. To accomplish that we are going to use the `ionChange` output event for the `ion-checkbox` element. Add this `(ionChange)="edit(todo.key, todo.val)"` to the checkbox.
 
 ```
<ion-label>{{todo.val.title}}</ion-label>
<ion-checkbox [(ngModel)]="todo.val.completed" (ionChange)="edit(todo.key, todo.val)"> {{todo.val.title}} </ion-checkbox>
```
If you remember from part 1 the perenthsis means that the data is bond one way and that is from the view to the data source. So when this event happens it will send data to our edit function.

Thats it! Now our todos can be updated as much as you want. 

NOTE: If you want you could also make an other page to edit the name of the todo. Leave a comment if you would like me to cover that change in this post.

## Athentication

##### Note: I will be covering only firebase auth and not adjusting the database rules (Authorization). That will be covered in a different post. 

Authentication is really made up of two parts the actual authenticaton ie. log the user in, setup there, account, etc. and authorization ie. what does the user have access to. Firebase has a great way to handle both by providing an authentication platform and by giving system administrators the ability to setup rules about what data different users can see. 

To start we need to go to our firebase console and click **Auth** on the left menu. From there click **Setup Up Sign-in Method**. Lets choose two. I went with Email & Password and Google. Other require a key and secret but should be similar in implementation. 

##### Note: Google does require a SHA1 for use on android. If you have trouble with that on your own please leave a comment and I will try to help.

Now lets create a new proivder for authentication called auth.

`ionic g provider Auth`

Remove all of the pre-generated `http` code and replace it with a class member for the firebase authentication reference. 
```
import {Injectable} from '@angular/core';

/*
  Generated class for the Auth provider.

  See https://angular.io/docs/ts/latest/guide/dependency-injection.html
  for more info on providers and Angular 2 DI.
*/
@Injectable()
export class Auth {
  auth: any = null;
  constructor() {
    this.auth = firebase.auth();
  }
}
```

Very basic. Our users need to do three things 1. Login, 2. Sign up, 3. Sign out. So lets add three methods to our Auth class `login`,`signup`, and `logout`.

#### Signup
We will start with signup. After all you can't login if there are no users. In our sign method we will take in two parameters `email` and `password`. We will call the firebase `createUserWithEmailAndPassword` function which returns a promise. That promise will be passed straight back to the caller since the auth provider does not care about the success or failure. It acts as more of a wapper around firebase. 

```
signup(email: string, password: string)
{
  return this.auth.createUserWithEmailAndPassword(email, password)
}
```

#### Login
Since we chose email & password, and google our auth function needs to take in an email and a password. However, because of the google login method we are going to make them optional. To sign in using email and password we will call `signInWithEmailAndPassword` giving it our email and password. 
```
login(email, password)
{
  if(email && password)
  {
    this.auth.signInWithEmailAndPassword(email, password)
        .catch((err) => {
          console.error('Login Failure', err);
        })
  }
  else
  {
    
  }
}
```

Our google authentication provider has two ways we can sign the user in. We could use the popup method to display a login overlay or we could use a redirect. Firebase docs recommend using the redirect for mobile and since we are using ionic that is exactly what we will do. But first we will need to add a reference to our google provider by adding a member variable to our auth class, and initializing it in the constructor. 
```
export class Auth {
  auth: any = null;
  google: any = null;

  constructor() {
    this.auth = firebase.auth();
    this.google = new firebase.auth.GoogleAuthProvider();
  }
```
In our `login` method after we check that we do not have a username and password we will try to authenticate with google. We will do this by calling `signInWithRedirect` and pass it our google provider `this.google`.  

```
 login(email, password)
  {
    if(email && password)
    {
      this.auth.signInWithEmailAndPassword(email, password)
        .catch((err) => {
          console.error('Login Failure', err);
        })
    }
    else
    {
      this.auth.signInWithRedirect(this.google)
    }
  }
 ```
Notice we did not return a promise here. Thats because the redirect does not return on and we will have to catch its output in a listener function after our page reloads.


#### Logout
Logout will be the simplest of the three methods. 

```
logout()
{
  return this.auth.signOut();
}
```
#### User State
Next we are going to want to keep track of the user state. We will do this by setting up a observer in our auth service. Firebase provides a function for just that case called `onAuthStateChanged`. We will use this to check if the user is currently logged in or not. 

First lets add a new member to our Auth class called `currentUser`. `currentUser: any = null; ` then in our constructor we will add the obersver setup.
```
firebase.auth().onAuthStateChanged((user) => {
  if (user) {
    this.currentUser = user;
  }
  else
  {
    this.currentUser = null;
  }
});
```

and we will add property to get that user.
```
get user()
{
return this.currentUser;
}
```
Our final auth provider looks like this: 

```
import {Injectable} from '@angular/core';

/*
  Generated class for the Auth provider.

  See https://angular.io/docs/ts/latest/guide/dependency-injection.html
  for more info on providers and Angular 2 DI.
*/
@Injectable()
export class Auth {
  auth: any = null;
  google: any = null;
  currentUser: any = null;

  constructor() {
    this.auth = firebase.auth();
    this.google = new firebase.auth.GoogleAuthProvider();
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.currentUser = user;
      }
      else
      {
        this.currentUser = null;
      }
    });
  }
  get user()
  {
    return this.currentUser;
  }
  login(email, password)
  {
    if(email && password)
    {
      this.auth.signInWithEmailAndPassword(email, password)
        .catch((err) => {
          console.error('Login Failure', err);
        })
    }
    else
    {
      this.auth.signInWithRedirect(this.google)
    }

  }
  signup(email, password)
  {
    return this.auth.createUserWithEmailAndPassword(email, password)
  }
  logout()
  {
    return this.auth.signOut();
  }
}

```
#### Login Page

Now we will generate our login page. This page will handle both login and sighup.
Generate a new page with the ionic cli.

`ionic g page login`

Then add an input group for our email and password. Like with our todos this will be an `ion-list` containing `ion-items`. Then at the bottom we will have three buttons one for our email and password login, one for our google login, and one for signing up.
```
<!--
  Generated template for the LoginPage page.

  See http://ionicframework.com/docs/v2/components/#navigation for more info on
  Ionic pages and navigation.
-->
<ion-navbar *navbar>
  <ion-title>login</ion-title>
</ion-navbar>

<ion-content padding class="login">
  <ion-list>
    <ion-item>
      <ion-label> Email</ion-label>
      <ion-input type="email" value="email"></ion-input>
    </ion-item>
    <ion-item>
      <ion-label>Password</ion-label>
      <ion-input type="password" value="password"></ion-input>
    </ion-item>
  </ion-list>
  <button (click)="login(email, password)"> Login</button>
  <button (click)="loginGoogle()">Google</button>
  <button (click)="signup(email, password)">Sign Up</button>
</ion-content>
```

In our login.ts file we now need to make our login methods and handle the transition from login page to todo list page. So import our home page, ionic loading controller, and our Auth service. 
```
import {Component} from '@angular/core';
import {NavController, Loading} from 'ionic-angular';
import {HomePage} from '../home/home';
import {Auth} from '../../providers/auth/auth';
```
 Don't forget to add the auth service to the constuctor.
 ```
 constructor(public nav: NavController, public _auth: Auth) {}
 ```

Then we will add three methods `login`, `loginGoogle`, and `signup`. The reason we are making these methods instead of just calling auth out right is because we need to give some user feed back and handle moving pages. 
```
login(email, password)
```






