![](http://i.imgur.com/EfDAYzR.png)

## Intro
This post is the fifth post in a series of posts which will teach you how to take advantage of your web development knowledge in building hybrid mobile applications for iOS and Android. The series comprises of the following published tutorials:

+ [How to get started with Ionic framework on Windows and Mac](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows)
+ [How to create a calculator application with Ionic framework by using Ionic Creator for UI](http://tutorials.pluralsight.com/review/how-to-create-a-calculator-application-with-ionic-framework-by-using-ionic-creator-for-ui/article.md)
+ [How to polish, create icons and splash screen images, add ads, share and test our calculator application](http://tutorials.pluralsight.com/review/how-to-polish-create-icons-and-splash-screen-images-add-ads-share-and-test-our-calculator-application/article.md)
+ [How to publish our calculator application to the Apple's App Store and Google's Play Store](http://tutorials.pluralsight.com/review/how-to-publish-our-calculator-application-to-the-apple-s-app-store-and-google-s-play-store/article.md)

In the series published so far, we showed how to create an application starting with an idea, through creating a prototype, implementing the application and finally publishing it in the stores. In this and future tutorials, we'll go over some popular existing applications and show how we would implement them with Ionic. At this point, **I encourage you to share your wishes and ideas** about what kind of an app would you like to see built step by step. Remember, I'm here to help you get the best of Ionic framework so don't hesitate to ask.

## TL;DR

In this tutorial, you will learn how the app [ShoppingTODO](http://shopping-todo.com/) (published both in [Apple's App Store](https://itunes.apple.com/us/app/shopping-todo/id1084227361?ls=1&mt=8) and [Google's Play Store](https://play.google.com/store/apps/details?id=com.shopping_todo)) was built. You'll get access to the full source code on [Github](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial) and I'll go through each and every file and explain all the bits and peaces that were needed for it to work.

You will learn how to use [Firebase](https://www.firebase.com/) as your backend and also how to incorporate the Material design. The app will have features like **list sharing**, **budget tracking**, and **live updates**.

You can even try the app in your browser [here](http://shopping-todo.com/mobile/index.html#/app/login). Yes, with Ionic, it's easy to take the same code base and deploy to an actual mobile App store and a standalone website application.

For actual deployment to the App Stores, please take a look at [the 4th tutorial](http://tutorials.pluralsight.com/review/how-to-publish-our-calculator-application-to-the-apple-s-app-store-and-google-s-play-store/article.md) which covers the exacts steps you need to take in order to publish a finished application to the Apple's App Store and Google's Play Store.

# It all starts with an idea
As you already learned through this series, it all starts with an idea. The idea which, purposefully, serves to solve a problem that we may have. Our problem is easy to define - we tend to forget things which we have to buy at the store. Even more so, we don't like getting tricked into buying stuff we don't need, and then having buyers remorse the second we get home. 

How many times you went into the store with a paper full of "items to buy"? And how many times you were crossing it off with your pen (given the presumption you didn't forget the pen at home) while in the other hand using your smartphone to chat with your friend. So, consequently, why not make an app for it?

The app will let you enter things that you have to buy. You would prepare this list at home (or your wife would prepare it for you, in case you wish to have a 'shared' list), and then once in the store, you would be able to enter exact prices and amounts of the stuff you actually bought. The most useful thing when hacking the budget will be its immediate display of the total price. Also, live updates will enable "just in time" edits so that the person you share the lists with doesn't have to call you on your phone to let you know that you should add that "one more thing".

## Design
The way were going to approach this implementation is that we'll try to reuse as much as open source help as possible. That's why we'll use an open source theme. Since Material design is the rather new cool thing now, we'll use the following [open source template](https://github.com/zachsoft/Ionic-Material). More specifically, we will only use the demo example.

## Finished project screens
The finished app, which you can take a look at [here](http://shopping-todo.com/), has the following screens:

+ Login - the app offers Facebook and Google login, and it even has the DEMO mode where you can test the app without needing to log in

<img src="http://i.imgur.com/6DqqUlY.jpg" style="width: 300px" />

+ Home screen - where you can add as many projects (we call them `groups` in the code) as you like

<img src="http://i.imgur.com/cnD3Wbv.jpg" style="width: 300px" />

+ Project screen - shows a list of items that need to be bought and those that are already bought, with a view of the price of bought/to-buy items

<img src="http://i.imgur.com/or5l1ll.jpg" style="width: 300px" />

+ Adding items on the project screen

<img src="http://i.imgur.com/yE7IecW.jpg" style="width: 300px" />

+ Adding users to your project

<img src="http://i.imgur.com/k9olacL.jpg" style="width: 300px" />

## Finished project code
You can clone the project and get it running locally by doing the following:

+ clone the code from [Github](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial)
+ `cd` into the new folder
+ run `ionic serve`
+ you should see the login screen (shown above) in your browser

In case you want to run it on your phone, you need to make sure the plugins listed below are installed (after adding the platform with `ionic platform add ios` or `ionic platform add android`). You can list the installed plugins by executing `ionic plugin list` in your project root directory (where your **ionic.project** file resides).

+ cordova-plugin-admobpro 2.11.1 "AdMob Plugin Pro"
+ cordova-plugin-extension 1.2.9 "Cordova Plugin Extension"
+ cordova-plugin-inappbrowser 1.2.1 "InAppBrowser"
+ cordova-plugin-splashscreen 3.2.0 "Splashscreen"
+ cordova-plugin-statusbar 2.1.1 "StatusBar"
+ cordova-plugin-whitelist 1.2.1 "Whitelist"
+ ionic-plugin-keyboard 1.0.8 "Keyboard"

To install the missing plugins, here's a copy/paste installation shortcut (execute this in your Command prompt/Terminal once in the root of your project):

+ cordova plugin add cordova-plugin-admobpro
+ cordova plugin add cordova-plugin-extension
+ cordova plugin add cordova-plugin-inappbrowser
+ cordova plugin add cordova-plugin-splashscreen
+ cordova plugin add cordova-plugin-statusbar
+ cordova plugin add cordova-plugin-whitelist
+ cordova plugin add ionic-plugin-keyboard

Additionally, you can take a look at this project with your [Ionic View](http://view.ionic.io/) application (we mentioned this in the previous tutorials) by entering the code: **4f055b6d**. Or, as mentioned before, you can download it from the [App Store](https://itunes.apple.com/us/app/shopping-todo/id1084227361?ls=1&mt=8) or [Play Store](https://play.google.com/store/apps/details?id=com.shopping_todo) or even try it out [in your browser](http://shopping-todo.com/mobile/index.html).


## How does it work?
As I mentioned in the design section above, I used an awesome [Ionic Material](https://github.com/zachsoft/Ionic-Material) template as a basis for this project. I literally cloned the project with:

`git clone https://github.com/zachsoft/Ionic-Material`

and took out the `demo` folder for the basis of my project. You can do the same steps and once you're in the demo folder if you run `ionic serve --lab` you'll get the following screen:

![](http://i.imgur.com/5ArTdPb.jpg)

### Starting point

> From this point on forward, I will not be copy pasting the full source code which I'm referring to here. Nor will I explain every single line, since that would become too cumbersome (and, besides, you're already pretty familiar with Ionic framework by now if you've followed the series thus far). Instead, I will give you the link on Github to the exact file/fuction that I'm referring to and will only show those lines of code that are necessary to explain what's happening. Of course, if you would have any, whatsoever, questions please post them as comments on this post and I'll be happy to help.

From my experience, when trying to understand some Ionic framework code, I always go to `app.js` file first and check out the [config() function](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/app.js#L112)(_click to view on Github_) to see how the routes are set up. Looking at our example we can see that the first screen that shows up is at route `/app/login`:

`$urlRouterProvider.otherwise('/app/login');`

We can now check the definition of this state and see that it looks like this:

```
.state('app.login', {
    url: '/login',
    views: {
        'menuContent': {
            templateUrl: 'templates/login.html',
            controller: 'LoginCtrl'
        },
        'fabContent': {
            template: ''
        }
    }
})
```

From here we can conclude several things:

+ the login state is `app.login` and we can reference it by that string
+ the login url is `/login` and that's what you would see in your URL if you ran it in  your browser
+ the login has two views (`menuContent` nad `fabContent`)
+ HTML of the `menuContent` view is defined in the `templates/login.html` file
+ `fabContent` view hasn't got a template associated with it

Concerning the two views, it's how the author of this template envisioned this. And, in case you're wondering why the name "fab", it comes from the  Floating Action Button in [Material design documentation](https://www.google.com/design/spec/components/buttons-floating-action-button.html). Consequently, this is the view that will hold the FAB button (+ button in the bottom right corner on the app screen images above).

### login.html
So, let's now check out the login logic by first examining the [login.html](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/login.html)(_click to view on Github_) template. The only interesting part here is the `ng-click` on the login buttons which calls the `oauthLogin()` function (with a proper provider passed in - 'facebook' or 'google') and the call to `testAppWithoutLogin()` function which basically logs a user in with an existing demo account.

###Firebase
In this project, Firebase is used as app's backend, including data storage and user authentication. It features a real-time database which came very handy with the live updates feature, and it can also do static hosting. You can learn more about them (they have guides for a lot of platforms) [here](https://www.firebase.com/docs/).

### LoginCtrl
`LoginCtrl`, found in [js/controllers.js](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L92)(_click to view on Github_) file, is the controller that contains the logic for the `login.html` template.

This controller contains three functions:

+ [oauthLogin](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L130) - depending on the passed parameter it calls the corresponding function in the [Auth service](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/Auth.js)
+ [testAppWithoutLogin](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L166) - prepares the demo user object and calls the [validateUser](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L104) function that calls the `loginWithEmail` function in the aforementioned Auth service.

[Auth service](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/Auth.js) is basically a custom factory that via AngularFire's (Angular wrapper for Firebase) `$firebaseAuth` service calls functions like `$authWithOAuthPopup` and `$authWithPassword`.

One important thing to note in the Auth service is the [onAuth](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/Auth.js#L55) function which wraps the `$onAuth` function with `$timeout` so it processes in the digest loop. In case you're wondering, we're listening to this event in the [app.js](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/app.js#L83) file.

> You can [learn more about AngularFire](https://www.firebase.com/docs/web/libraries/angular/quickstart.html) from official Firebase documentation. More specifically, they have a [step by step tutorial for Ionic](https://www.firebase.com/docs/web/libraries/ionic/guide.html) as well, which you may want to follow in order to get a better grasp on the subject.
 
As for the social logins (Facebook and Google), you will have to create applications on their developer sites and you can follow the corresponding step by step official tutorials [here](https://www.firebase.com/docs/web/guide/login/facebook.html) and [here](https://www.firebase.com/docs/web/guide/login/google.html) to learn how to do that. Basic Firebase user authentication documentation can be found [here](https://www.firebase.com/docs/web/guide/user-auth.html).

As a quick tip on linking Facebook and Firebase; all you have to do is enable Facebook authentication in the settings and enter the appropriate App Id and Secret (which you'll get once you create a Facebook application).

![](http://i.imgur.com/gKBjZdK.jpg)

Additionally, probably the only part that you must not forget about is to set the proper OAuth redirect URI on the Facebook `Settings -> Advanced` tab:

![](http://i.imgur.com/Yu2liaH.jpg)

### User profile
Once the user logs in successfully we forward him to the `app.profile` state, as you can see [in the code](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/app.js#L63). Now, again we go back to the [app.js config part](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/app.js#L156) to see what is the route definition for this state:

```
.state('app.profile', {
    url: '/profile',
    views: {
        'menuContent': {
            templateUrl: 'templates/profile.html',
            controller: 'ProfileCtrl'
        },
        'fabContent': {
            templateUrl: 'templates/profileFabButtonAdd.html',
            controller: 'ProfileCtrlFabButton'
        }
    }
})
```

From here we can conclude several things:

+ the profile state is `app.profile` and we can reference it by that string
+ the profile url is `/profile` and that's what you would see in your URL if you ran it in  your browser
+ the profile has two views `menuContent` and `fabContent`
+ HTML of the `menuContent` view is defined in the `templates/profile.html` file. Also, this template has a corresponding `ProfileCtrl` controller
+ HTML of the `fabContent` view is defined in the `templates/profileFabButtonAdd.html` file. Also, this template has a corresponding `ProfileCtrlFabButton` controller

### profile.html
The [profile.html](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/profile.html)(_click to view on Github_) template displays logged in user's profile image, name, and email. Also, it lists his existing projects (if he has any) with the [ng-repeat directive](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/profile.html#L34)(_click to view on Github_).

Inside the ng-repeat directive, we are displaying the project image and name. By using the `<ion-option-button>` directive, we're adding support for the "swipe left" gesture which reveals `Remove` and `Edit` buttons, which in turn call the corresponding [Remove](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L451) and [Edit](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L502) functions in the [ProfileCtrl](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L402) via the `ng-click` directive.

### profileFabButtonAdd.html
[This file](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/profileFabButtonAdd.html) contains a simple button which is used for adding the new project. The `ng-click` directive calls the [add](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L578) function, defined in [ProfileCtrlFabButton controller](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L568).

The `add` function basically shows the popup (via `$ionicPopup` service) and, after the user enters the new project name and confirms, it calls the `GroupAdd` function defined on the [Groups factory](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L55) (in the file named [data.js](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js)).

### Adding items to the list
Once the user creates a new project (list) and clicks on this list in his profile he will be forwarded to the `app.list({id: g.id, ime: g.name})` state, as you can see [in the code](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/profile.html#L34). Now, again we go back to the [app.js config part](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/app.js#L129) to see what is the route definition for this state:

```
.state('app.list', {
    url: '/list/:id/:ime',
    views: {
        'menuContent': {
            templateUrl: 'templates/list.html',
            controller: 'ListCtrl'
        },
        'fabContent': {
            templateUrl: 'templates/listFabButtonAdd.html',
            controller: 'ListCtrlFabButton'
        }
    }
})
```

From here we can conclude several things:

+ the list state is `app.list` and we can reference it by that string
+ the list url is `/list` with additional two url parameters `id` and `ime` (in case you're wondering that's Croatian for 'name' - _yeah, inconsistency isn't a virtue_)
+ the list has two views: `menuContent` and `fabContent`
+ HTML of the `menuContent` view is defined in the `templates/list.html` file. Also, this template has a corresponding `listCtrl` controller
+ HTML of the `fabContent` view is defined in the `templates/listFabButtonAdd.html` file. Also, this template has a corresponding `ListCtrlFabButton` controller

### list.html
The [list.html](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/list.html)(_click to view on Github_) template displays the items (along with their name, price, and quantity) added to the list.

Inside the ng-repeat directive, we are displaying the item name, price, and quantity, ordered by `isCompleted` property which indicates that the item has been "bought".

Each list item can be [deleted](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/list.html#L33) and marked as ["bought"](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/list.html#L43) by clicking the corresponding button, which in turn then call the corresponding [Remove](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L230) and [CheckOrUncheck](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L238) functions in the [ListCtrl](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L176) via the `ng-click` directive.

These two controller functions call the functions defined in the `Groups` factory which then deals with Firebase in order to delete the item or mark it as completed. Important to note is the usage of [transaction](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L240) operation on the returned item, which basically secures that some change happens in a transaction (which, from database theory means - either the data will be saved, or it will not be saved at all).

`transaction()` returns stored data so you can [manipulate](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L243) it and change the data that is [returned](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L245). You can learn more about transaction function from the [official documentation](https://www.firebase.com/docs/web/api/firebase/transaction.html).

### Inviting users to your list
Additionally, this template has [one more button](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/list.html#L4) which is used for adding a new user to the list of users who share this concrete list. The `ng-click` directive calls the [addUserToGroup](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L285) function, defined in [ListCtrl controller](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#176).

This function shows a popup with an username input field and once the user enters his friend's email and confirms it, the controller will call the external service on the server which will actually send an email to this person notifying him that his friend added him to the list. 

Also, this list will appear in their project list once they log in to their account. You can, of course, comment [this code](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L338) out or place your own API for this.

### listFabButtonAdd.html
[This file](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/templates/listFabButtonAdd.html) contains a simple button which is used for adding the new item to the list. The `ng-click` directive calls the [add](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L357) function, defined in [ListCtrlFabButton controller](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L351).

The `add` function basically shows the popup (via `$ionicPopup` service) and, after the user enters the new item name, price and quantity and confirms, it calls the `GroupItemAdd` function defined on the [Groups factory](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L219) (in [data.js file](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js) file).

### Groups factory
Let us take a moment now and go over the functions in the [Groups factory](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L55):

+ [Groups.byUser](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L63) - this whole function is done in chained promises. We first get all groups that are stored in array in the [user object](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L64). Then we [iterate](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L68) through this list and [create an array](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L70) with promises for all [groups in the list](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L71). Next, we  get [items](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L75) for that group (this was added initially because we thought of adding item list under the group name). After that we [iterate through members array](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L93) in group and [create an array of promises](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L95) which we [resolve](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L104) in the end with [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all). After members are resolved we return an [array of group promises](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L115) and [resolve this whole array in the end](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L118).

+ [Groups.Group](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L122) - returns a promise for group data

+ [Groups.GroupAdd](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L126) - first we need to [create a reference](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L129) with [push()](https://www.firebase.com/docs/web/api/firebase/push.html) which contains new unique key for that group. Then we fill the [object of that group](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L133), and with reference that we got from `push()` we [write this object](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L145) with [set()](https://www.firebase.com/docs/web/api/firebase/set.html) function.

+ [Groups.GroupAddMember](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L155) - first we [add a group key to user object](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L161) and then we [add the user to the group](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L168) object under members.

+ [Groups.GroupRemoveMember](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L179) - here we use [remove()](https://www.firebase.com/docs/web/api/firebase/remove.html) function to remove a member from group object and return a promise which this function returns.

+ [Groups.GroupRemove](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L188) - before we can remove the group we need to [clean the items](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L192) associated with this group. In order to do so, we can get the reference for these items through [GroupItems](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L211) function. After we successfully delete this group then we [remove the group](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L196) from groups list and [remove group key from user object](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L200).

+ [Groups.GroupItems](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L211) - returns reference to the list of group items.

+ [Groups.GroupItem](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L215)  - returns the reference to one record (by item key) from list of items for some group.

+ [Groups.GroupItemAdd](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L219) - just like when we were adding a new group, we need to first call [push()](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L222) on reference that we got from `Groups.GroupItems` in order to get the reference to the new group, and unique key for that group. Then we fill the [object of that item](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L226), and with the reference that we got from `push()` [write this object](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L238) with [set()](https://www.firebase.com/docs/web/api/firebase/set.html) function.

+ [Groups.GroupItemRemove](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L252)  - to [remove item from group item list](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L252) we call `remove()` function on the reference that we got from `Groups.GroupItems`

+ [Groups.setCurrentGroupId](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L264) and [Groups.getCurrentGroupId](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L268) - these functions are used to set and get [currentGroupId](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/data.js#L59) that is used in [ListCtrl](https://github.com/Hitman666/IonicShoppingTodoFirebaseMaterial/blob/master/www/js/controllers.js#L176)

### Firebase access rules
What's left to show is how the Firebase rules (found in the `Dashboard -> Security & Rules` in your Firebase admin dashboard) are set up. You will find comments on how and what each rule does in the comments (parts marked with `//`)

```
{
  "rules": {
    // By default, make all data private unless specified otherwise.
    ".read": false,
    ".write": false,
    
    "users": {
      "$userId": {
        // Only authenticated user can read and make chanages on it's own data (auth.id equal to $userId)
        ".read": "(auth != null) && (auth.uid === $userId)",
        ".write": "(auth != null) && (auth.uid === $userId)",
        ".validate": "($userId === newData.child('id').val()) && (newData.hasChildren(['id','provider']))",
        // Write only if group exists
        "groups": {
          "$groupId": {
            ".validate": "(root.child('groups').hasChild($groupId) && !root.child('users').child($userId).child('groups').hasChild($groupId))"
          }
        }
      }
    },
    "users-metadata": {
      // A list of users and their associated metadata, which can be updated by the single user and read by authenticated users
      "$userId": {
        ".read": "(auth != null) && (root.child('users').hasChild(auth.uid))",
        ".write": "(auth != null) && (auth.uid === $userId)",
        ".validate": "($userId === newData.child('id').val() && root.child('users').hasChild($userId)) && (newData.hasChildren(['id','name']))"
      }
    },
    "users-byemail": {
      // A list that contains email keys, which can be updated and read by authenticated users
      ".read": "(auth != null) && (root.child('users').hasChild(auth.uid))",
      ".write": "(auth != null) && (root.child('users').hasChild(auth.uid))",
      "$userMailHash": {
        //Write only if users exists
        "user_id": {
          ".validate": "(root.child('users').hasChild(newData.val()) && newData.val() === auth.uid) || (newData.val() === data.val())"
        },
        "invites": {
          "$groupId": {
            ".read": "(auth != null) && (auth.uid === data.child('fromUserId').val())",
            ".write": "(auth != null) && (!data.exists() || (root.child('groups').hasChild(newData.child('groupId').val()) && newData.child('fromUserId').val() === auth.uid))",
            ".validate": "newData.hasChildren(['fromUserId','groupId']) && (newData.child('groupId').val() === $groupId)"
          }
        }
      }
    },
    
    "groups": {
      // A list with groups and their metadata, which can be updated and readed by authenticated users
      "$groupId": {
        ".read": "(auth != null) && (root.child('users').hasChild(auth.uid))",
        ".write": "(auth != null) && (root.child('users').hasChild(auth.uid))",
        ".validate": "((auth.uid === newData.child('owner').val() || newData.child('owner').val() === data.child('owner').val()) && newData.hasChildren(['id','name','owner']) && $groupId === newData.child('id').val())",
        // Write only if user exists
        "members": {
          "$userId": {
            ".validate": "(auth != null) && (root.child('users').hasChild($userId))"
          }
        }
      }
    },
    "group-items": {
      "$groupId": {
        // A list of items in group, which can be updated and readed by authenticated users that are members of some group
        ".read": "(auth != null) && (root.child('groups').child($groupId).child('members').hasChild(auth.uid))",
        ".write": "(auth != null) && (root.child('groups').child($groupId).child('members').hasChild(auth.uid))",
        "$itemId": {
          ".validate": "(newData.hasChildren(['id','created','createdByUserId','groupId','isCompleted','name','updated','price','quantity']) && $itemId === newData.child('id').val())",
          "createdByUserId": {
            ".validate": "(newData.val() === auth.uid || data.val() === newData.val()) && (root.child('users').hasChild(newData.val()))"
          },
          "groupId": {
            ".validate": "(root.child('groups').hasChild(newData.val()))"
          },
          "price": {
            ".validate": "(newData.isNumber())"
          },
          "quantity": {
            ".validate": "(newData.isNumber())"
          }
        }
      }
    }
  }
}
```

## Conclusion
This tutorial may not have been as detailed (or as long - yes, I'm looking at you 3rd 7k+ word tutorial :)) as the previous ones and that's basically due to the fact that then the whole tutorial would be more than 10k words long, and we all know that not everyone is prone to reading tutorial essays. All in all, this is by far fully polished and optimized, so I encourage you to take it a step further.

## Free to be free
I hope this tutorial, and the accompanying project helped you and that you've seen the potential that Firebase offers and that you'll create something great with it yourself.

As I said in the introduction I encourage you to share your wishes and ideas about what kind of an app would you like to see built and then explained. Remember, I'm here to help you get the best of Ionic framework so don't hesitate to ask.

In the end I want to take a moment and say that you're free to use this project in any way you like. You can fork it, make it better, submit your version to app store, make money out of it - no problem, just make sure you send me a beer or two in case you pull it off ;)

## Credits
Friend of mine, Matija Lesar, was helping me with this application and was responsible for communication with Firebase via Angular services and Firebase data model. He is eager to learn new things and try new programming languages but his real passion are databases and backend services. You can look him up on his [personal web page](http://ffox.com.hr).

## How to get help with Ionic framework
Few of the resources that proved indispensable when I was learning about Ionic:

+ For a quick framework reference, I'm suggesting the [official documentation](http://ionicframework.com/docs/), which is indeed very good.
+ If you're in search for a good book about Ionic, then you don't need to look further than [Ionic in Action: Hybrid Mobile Apps with Ionic and AngularJS](http://amzn.to/1EjwYNJ)
+ Of course, if you haven't, check out my posts in this series
+ One of the best resources on the net for programming related questions, about which you've no doubt heard, is [StackOverflow](http://stackoverflow.com/). You can view the specific Ionic tagged questions via [this link](http://stackoverflow.com/questions/tagged/ionic).
+ To get started with AngluarJS try [these](http://angular.codeschool.com/) [few](https://hackhands.com/finishing-Angular-TODO-application-deploying-production/) [resources](https://docs.angularjs.org/tutorial), not necessarily in that particular order.
+ Let me just kind of repeat myself by saying that if you have **any questions** about Ionic framework, or you had trouble following this tutorial (you couldn't install something), or you would like to suggest what kind of tutorials you would like to see regarding Ionic in the future, then please **share it in the comments** below. Also, you can reach me personally via @[HitmanHR](https://twitter.com/hitmanhr) or [my blog](http://www.nikola-breznjak.com/blog). I'll do my best to  answer all of your questions.
+ If, however, you favor "one on one" help, you can reach me [via HackHands](https://hackhands.com/dashboard/request/hitman666/preferred).