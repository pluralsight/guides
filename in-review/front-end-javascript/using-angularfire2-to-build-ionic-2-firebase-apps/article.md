# _Ionic 2_ + _AngularFire2_ = **Awesomeness**!

In this year's I/O google introduced the [new version of firebase](https://firebase.google.com/) and it came packed with goodies, from analytics integration to admob to monetize your apps, it has everything.

As soon as they announced the big update, [David East](https://twitter.com/_davideast) and [Jeff Cross](https://twitter.com/jeffbcross) started working really hard in making [AngularFire2](https://angularfire2.com/api/) compatible with the new services.

For [ionic framework](https://ionicframework.com) developers this means we get a really great library that integrates deeply with the framework, helping us create our apps easier and faster.

I have been using **AngularFire2** for over a month now and I find it fascinating how fast I'm building my apps now.

Today I want to share with you a guide on how to create an app using **Ionic 2 + AngularFire2** so you can make the jump and start building with that combo soon!

We are going to create a small app to keep track of our unpaid bills, this is something I made for my wife and me since we keep forgetting them :P

So with that intro I think it's time to get to business:

The first thing you want to do is to make sure you have the latest version of ionic installed in your machine, so go ahead and open your terminal and type:

```bash
  npm install -g ionic@beta cordova typings
```

What this does is installs the latest version of the Ionic CLI (you'll use it to create your projects) cordova (this is how you get that nice installable) and typings to work with TypeScript.

Once you have everything installed is time to create your app, go back to your terminal and navigate to wherever you want to create the app and create it, for me is in my development folder:

```bash
  cd Development
  ionic start billTracker blank --v2
  cd billTracker
```

That does a few things:

* **ionic start** creates the new app.
* **billTracker** is the name we are giving the app.
* **blank** tells the CLI we don't want to use any starter template, just a blank project.
* **--v2** tells the CLI to create an Ionic2 app instead of Ionic1 (_pretty soon this will be default_)

Now you have created a new app and are inside your app folder, it's time to install the dependencies we'll need, right now you'll install **firebase**, **angularfire2** and **typings for firebase**.

So go back to your terminal and inside the `billTracker` folder type:

```bash
  npm install angularfire2@2.0.0-beta.2 && firebase --save
  typings install --save --global firebase
```

The reason I'm using the `@2.0.0-beta.2` tag when installing angularfire2 is because I need it to work with Firebase's new console, if you leave the tag out it will work with the legacy console.

Now that we have everything installed it's time to open your app in your favorite text editor or IDE, I'm using atom so I just type `atom .` inside my app folder.

Go to `app/app.ts` and you should see something like this:

```javascript
import { Component } from '@angular/core';
import { Platform, ionicBootstrap } from 'ionic-angular';
import { StatusBar } from 'ionic-native';
import { HomePage } from './pages/home/home';

@Component({
  template: '<ion-nav [root]="rootPage"></ion-nav>'
})
export class MyApp {
  rootPage: any = HomePage;

  constructor(platform: Platform) {
    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      StatusBar.styleDefault();
    });
  }
}

ionicBootstrap(MyApp);

```

You need to do a couple of things here, first, at the top of the file you'll see were we import the components we need, go ahead and add an import for angularfire2 components:

```javascript
import { FIREBASE_PROVIDERS, defaultFirebase } from 'angularfire2';
```

After that you'll need to pass those to the `ionicBootstrap` that you'll find at the end of the file:

```javascript
ionicBootstrap(MyApp, [
  FIREBASE_PROVIDERS,
  defaultFirebase({
    apiKey: "AIzaSyCw1SKmz1Bu2ibncu5Bxd1sZgR1taV7ujM",
    authDomain: "aftutorial-ef306.firebaseapp.com",
    databaseURL: "https://aftutorial-ef306.firebaseio.com",
    storageBucket: "aftutorial-ef306.appspot.com",
  })
]);
```

You'll need to replace the `defaultFirebase` options with your own, you can use those for testing, those are from an app I made exclusively for this tutorial so no biggie there.

If you don't know where to get those, you'll need to go to the [Firebase Console](https://console.firebase.google.com/) and click on your app (_or create a new one_)

You'll see a welcome message like the one in the image below, just pick **Add Firebase to your web app** and it'll open a pop up with all the info for you to use.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f89ed33a-14db-444f-ab7a-e6c644a68489.png)


Now we are ready to start building, we are already connected to firebase so most configuration is done (_by now_)

The first thing we'll want to do is to add a button so our users can create new bills inside their app, so go to `app/pages/home/home.html` it should look like this:

```html
<ion-header>
  <ion-navbar>
    <ion-title>
      Ionic Blank
    </ion-title>
  </ion-navbar>
</ion-header>

<ion-content class="home">

</ion-content>
```

Nothing weird here, it's just the basic html, go ahead and change the page title and add a navbar button that sends you to a new page to create bills:

```html
<ion-header>
  <ion-navbar>
    <ion-title>
      My Bills
    </ion-title>
    <ion-buttons end>
      <button (click)="newBill()">
        <ion-icon name="add"></ion-icon>
      </button>
    </ion-buttons>
  </ion-navbar>
</ion-header>

<ion-content class="home">

</ion-content>
```

That will give you a nice **+** sign to the right of your navbar, and once the user clicks the sign it calls the `newBill()` function, so it's time to go to `home.ts` and create that function, first open the file, it will have the placeholder code:

```javascript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';

@Component({
  templateUrl: 'build/pages/home/home.html',
})
export class HomePage {
  constructor(public navCtrl: NavController) {
  }

}
```

We are going to send the user to a new page to create this bill, so we'll need to use the `NavController` to send our user there, but first, let's create the page.

I wanted to pause here and create the page to show you the coolest thing about **Ionic** its CLI, you can do a bunch of cool stuff with it, even generate placeholder code, just open your terminal and type:

`ionic g page BillCreate`

That little line of code right there creates a `bill-create` folder, with 3 files in it `bill-create.ts bill-create.html bill-create.scss` and they all have the boilerplate code already there, talk about your productivity boost!

We'll open those files later, for now we just needed to create them so we won't get any errors when creating the `newBill()` function.

So go back to `home.ts` and right after the `constructor()` create the new function:

```javascript
newBill(){
  this.navCtrl.push(BillCreatePage);
}
```

This will just take our user to the newly created page where we'll add the form to create a new bill.

So go to `pages/bill-create/bill-create.html` and delete everything in there, we'll replace it with this:

```html
<ion-header>
  <ion-navbar>
    <ion-title>New Bill</ion-title>
  </ion-navbar>
</ion-header>

<ion-content padding class="bill-create">
  <ion-item>
    <ion-label stacked>Bill Name</ion-label>
    <ion-input [(ngModel)]="name" type="text" placeholder="How do you identify this bill?"></ion-input>
  </ion-item>

  <ion-item>
    <ion-label stacked>Amount</ion-label>
    <ion-input [(ngModel)]="amount" type="number" placeholder="How much will you have to pay?"></ion-input>
  </ion-item>

  <ion-item>
    <ion-label>Due Date</ion-label>
    <ion-datetime displayFormat="D MMM, YY" pickerFormat="DD MMM YYYY" [(ngModel)]="dueDate"></ion-datetime>
  </ion-item>

  <button block (click)="createBill(name, amount, dueDate)">
    Create Bill
  </button>

</ion-content>
```

The first 2 items created are just regular inputs, you can see everything about them in the [ionic docs](http://ionicframework.com/docs/v2/components/#inputs). We are just adding an `[(ngModel)]` to them to hold the values.

The third item is a date-time item, this is why Ionic is so awesome, you add that input and it will add a beautifully designed date picker to your app, you can read more about it [here](http://ionicframework.com/docs/v2/components/#datetime)

And finally we add a button that calls the `createBill()` method passing the values of the inputs.

Now we need to go to `bill-create.ts` and create that function, open the file and you'll see this:

```javascript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';

@Component({
  templateUrl: 'build/pages/bill-create/bill-create.html',
})
export class BillCreatePage {
  constructor(public navCtrl: NavController) {
  }

}
```

We'll need to import AngularFire and our list observable to this file, so right below your last import statement type:

`import { AngularFire, FirebaseListObservable } from 'angularfire2';`

* **AngularFire** gives us the angularfire2 magic, where we can call lists, objects, auth methods, etc.
* **FirebaseListObservable** gives us the observable we'll be using, it will subscribe to the list and sync it in real time with our app.

Right before the constructor create a variable that will hold our list, if you are getting confused on where to do this don't worry, I'll show you the completed file later.

`billList: FirebaseListObservable<any>;`

And inject `AngularFire` into your constructor.

```javascript
constructor(public navCtrl: NavController, public af: AngularFire) {}
```

Now create a binding to the list inside the constructor:

`this.billList = af.database.list('/bills');`

This will keep `billList` synced with our Firebase real-time database on the 
`/bills` node.

To add new bills to that list we just need to use angularfire's `push()` method:

```javascript
createBill(name, amount, dueDate) {
  this.billList.push({
    name: name,
    amount: amount,
    dueDate: dueDate,
    paid: false
  }).then( newBill => {
    this.navCtrl.pop();
  }, error => {
    console.log(error);
  });
}
```

* `this.billList.push()` is adding a new bill to the list passing the values the method is receiving and an extra property called `paid` set to `false`
* `.then()` function happens after the bill is added to the list and it does 2 things:
  - First it goes back one page (_to the Home Page_) after creating the bill.
  - Second, if any error occurs on the process it's going to log it to the console.

The final version of the file should look like this:

```javascript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { AngularFire, FirebaseListObservable } from 'angularfire2';

@Component({
  templateUrl: 'build/pages/bill-create/bill-create.html',
})
export class BillCreatePage {
  billList: FirebaseListObservable<any>;
  constructor(public navCtrl: NavController, public af: AngularFire) {
    this.billList = af.database.list('/bills');
  }

  createBill(name, amount, dueDate) {
    this.billList.push({
      name: name,
      amount: amount,
      dueDate: dueDate,
      paid: false
    }).then( newBill => {
      this.navCtrl.pop();
    }, error => {
      console.log(error);
    });
  }

}
```

Ok so we can create new bills now, is that it? Nope, now we are going to list those bills in the Home page, for that go ahead and open `home.ts` and import AngularFire, inject it to the controller and bind it to a list:

```javascript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { BillCreatePage } from '../bill-create/bill-create';
import { AngularFire, FirebaseListObservable } from 'angularfire2';

@Component({
  templateUrl: 'build/pages/home/home.html',
})
export class HomePage {
  billList: FirebaseListObservable<any>;
  constructor(public navCtrl: NavController, public af: AngularFire, public alertCtrl: AlertController) {
    this.billList = af.database.list('/bills');
  }

  newBill(){
    this.navCtrl.push(BillCreatePage);
  }

}
```

We're going to go to `home.html` and create a couple of cards to show both **paid** and **unpaid** bills, but since I want to format the Due Date a bit I'm going to need a little help.

You see, angular2 pipes (_that's what we use to format dates_) break when we are using Ionic to build mobile apps, so we're going to use a custom pipe from [angular2-moment](https://github.com/urish/angular2-moment)

Open your terminal and type:

```bash
npm install --save angular2-moment
typings install --save moment
```

This will install both the module and its typings, you can read the docs [here](https://github.com/urish/angular2-moment#usage) but we'll only use a small part of it.

Import the pipe we'll need:

`import { DateFormatPipe } from 'angular2-moment';`

Now inside our `@Component` declaration we need to express we'll use that pipe:

```javascript
@Component({
  templateUrl: 'build/pages/home/home.html',
  selector: 'app',
  pipes: [DateFormatPipe]
})
```

And that's it, we can go to `home.html` to use it, just open the file and inside the `ion-content` create a card to hold our **unpaid** bills:

```html
<ion-card>
  <ion-card-header>
    Pending Bills
  </ion-card-header>

  <ion-list>
    <ion-item text-wrap *ngFor="let bill of billList | async" (click)="promptPayment(bill.$key)"
      [class.hide]="bill.paid == true">
      <ion-icon name="timer" danger item-left></ion-icon>
      <h2>{{bill.name}}</h2>
      <h3>Total: <strong>${{bill.amount}}</strong></h3>
      <p>Pay before: <strong>{{bill.dueDate | amDateFormat: 'LL'}}</strong></p>
    </ion-item>
  </ion-list>
</ion-card>
```

There are a few things going on here:

* `*ngFor="let bill of billList | async"` tells our item to repeat for every bill it finds on our billList.
* `(click)="promptPayment(bill.$key)` tells our system to call that function when users tap on the item. (_we'll create that function in a minute_).
* `[class.hide]="bill.paid == true"` if the bill is already paid, add the hide css class, which will hide the item from the list.

And after that we are creating a regular item and binding the `bill` properties to it.

See this line `<p>Pay before: <strong>{{bill.dueDate | amDateFormat: 'LL'}}</strong></p>` this is where we are using angular2-moment to format the date.

Right after that card we'll create another card, this time to hold the list of the bills our user already paid:

```html
<ion-card>
  <ion-card-header>
    Paid Bills
  </ion-card-header>

  <ion-list>
    <ion-item text-wrap *ngFor="let bill of billList | async"
      [class.hide]="bill.paid == false">
      <ion-icon name="checkmark-circle" favorite item-left></ion-icon>
      <h2>{{bill.name}}</h2>
      <h3>Total: <strong>${{bill.amount}}</strong></h3>
    </ion-item>
  </ion-list>
</ion-card>
```

It's almost the same as before, we are just adding the hide class to the ones that haven't been paid yet.

Ok so remember when we said we'd create the `promptPayment()` function in a minute? It's that time now, go to `home.ts` and import AllertController, we'll need this to create a prompt:

`import { NavController, AlertController } from 'ionic-angular';`

Now it's just creating the function:

```javascript
promptPayment(billId: string) {
  let alert = this.alertCtrl.create({
    message: "Mark as paid?",
    buttons: [
      {
        text: 'Cancel',
      },
      {
        text: 'Mark as Paid',
        handler: data => {
          this.billList.update(billId, { paid: true });
        }
      }
    ]
  });
  alert.present();
}
```

The code is really easy to follow, we are creating a prompt that's going to ask the user: _Mark as paid?_ and our user is going to get 2 buttons:

* Cancel does nothing.
* Mark as Paid will call `this.billList.update(billId, { paid: true });` and this is going to take the billId we're sending from the html and update that bill with AngularFire's `update()` method, piece of cake!

There you have it, a fully functional app running on the latest version of **Ionic Framework** and **Firebase**

If you want you can download the entire source code for this app from [github](https://github.com/javebratt/bill-reminder)

And I also send out a tutorial every Thursday on Ionic + Firebase, you can join [here](http://javebratt.com/become-developer/) to make sure you start getting them :-)

If there's anything I can help you with just ask, I'm [@javebratt](https://twitter.com/javebratt)