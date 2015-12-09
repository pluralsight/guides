Promises are a relatively new way of handling asynchronous operations in a readable, composable and robust way. With the exception of IE11, [they are built into all modern browsers](http://caniuse.com/#feat=promises).

Ember.js is a client-side MVC framework for building rich browser applications which has "all parts included". It embraces the latest web technologies while maintaining support for browsers as far back as IE8\. Its official tool to develop Ember applications, ember-cli, supports [ES6 modules](http://www.2ality.com/2014/09/es6-modules-final.html) and [esnext](https://github.com/esnext/esnext). So it should come as no surprise that Ember chooses to use promises as a state-of-the-art solution for handling asynchronicity, and uses them in various ingenious ways.

In this post, I'll pull apart the curtains and show a few instances of how Ember leverages promises. Let's first start with brief intros to promises and Ember.js, to help make the examples clearer.

## Promises run-down

_If you know what promises are, you can skip ahead to the [next section](#ember-js-rush-down)_

A promise is a construct that "eventually" (asynchronously) yields a value. It can either "resolve" or "reject" which indicate success or failure, respectively. Once either of these results is reached, a promise cannot change its state, or the value it produced. This immutable nature makes it a robust tool for managing state asynchronously. Also, as several consumers can be attached to the same promise, they are guaranteed to see the same value and state.

The Promises A+ spec has several implementations, of which [Q.js](https://github.com/kriskowal/q), [Bluebird](https://github.com/petkaantonov/bluebird) and [RSVP.js](https://github.com/tildeio/rsvp.js/) are the most popular.

Ember chose RSVP.js so that's what I'm going to use in the following examples. To create a promise in RSVP, both a resolve and a reject callback must be passed in the constructor:

```javascript
var promise = new RSVP.Promise(function(resolve, reject)) {
  // call 'resolve' with the value if the operation is successful
  // call 'reject' with the error reason if the operation is unsuccessful
})

```

The way to interact with a promise is through its `then` method. The first argument (the fulfillment handler) attached to the promise gets called if the promise resolves with success. Conversely, the second argument is the rejection handler which gets called if the promise resolution resulted in rejection. The value that is passed is the reason the promise was rejected - e.g it can be an Exception object or a simple string.

The classic example is issuing an XHR to a backend:

```javascript
var request = new Ember.RSVP.Promise(function(resolve, reject) {
  Ember.$.ajax('/countries', {
    success: function(response) {
      resolve(response);
    },
    error: function(reason) {
      reject(reason);
    }
});

request.then(function(response) {
  // e.g render template from response
}, function(error) {
  // handle error (show error message, retry, etc.)
});

```

For the sake of understanding this post, that is as much as we need to know about promises for now. If you would like to dig deeper, you can read [a tutorial on html5rocks.com](http://www.html5rocks.com/en/tutorials/es6/promises/) or [a lengthier introduction](http://www.toptal.com/javascript/javascript-promises) written by yours truly.

## Ember.js rush-down

_If you know Ember.js, you can skip ahead to the [next section](#the-ember-run-loop)_

Ember.js is an "all-parts included" framework for developing browser applications. Its slogan states that it is for building "ambitious" web applications and this is not just a marketing gimmick. It really has all the building blocks that you need. Furthermore, it is very opinionated, which means that if you follow the Ember way (and learn what that way is), you will be productive most of the time.

It is an impossible mission to introduce Ember in a few paragraphs as I did with promises above, even on a superficial level. Instead, I will focus on explaining just enough to be able to appreciate the ingenuity of the ways Ember.js leverages promises.

## The Ember run loop

_If you know the basics of the run loop in Ember, you can skip ahead to the [next section](#ember-and-promises-sitting-in-a-tree)_

The run loop aims to reduce the amount of work done by batching tasks together in queues. To give you an example, changing the DOM is slow and thus reducing the number of times it needs to be done increases performance. Without the run loop an element that is shifted upwards 5 px twice will modify the DOM twice. In a run loop enabled environment, the DOM gets modified just once and the node is shifted upward 10px.

The run loop is composed of queues and each task is put into one of these queues depending on its type. The queues have priority and each time the run loop looks for a queue to process, it picks the one with the highest priority that has pending tasks in it.

Here are the queues in Ember in order of priority (accessible by typing `Ember.run.queues` in the console):
["sync", "actions", "routerTransitions", "render", "afterRender", "destroy"]

"sync" has the highest priority, "destroy" has the lowest. Promise resolution happens in "actions" and so has quite high priority.

## Ember and promises, sitting in a tree

Now that we are able to appreciate how Ember makes use of promises, let's see the actual scenarios and how they work under the hood.

### Start rendering the view when all data is in

One of the distinctive features of Ember.js is how much relevance it assigns to the router. The principle behind this decision is that "the URL is the UI of the web", and that anything that works with the classic, "100% server side" web development should continue working with Ember. This is succinctly summarized in the motto, "Don't break the Back button".

When the user enters `http://orgnzr.com/groups/1/members`, the router starts matching the path segments against the defined routes. It invokes the model hook for each matching route to fetch the data that will be used to render a part of the page.

The first match is probably the `groups` route that sends an ajax request to get all the groups, then for the `group` route a single group is retrieved. Finally, the members of that group are fetched and drawn on screen. (There are several techniques that can be used to reduce the number of requests but that is beyond the scope of this post.)

To see this in code, the model hook for the `groups` route might look something like this:

```javascript
// app/routes/groups.js
export default Ember.Route.extend({
  model: function() {
    return $.getJSON('/groups.json');
  }
});

```

If a promise is returned from the model hook (like in the above case), the rendering of the part of the screen that belongs to the route does not start until the promise is resolved.

In order to comprehend how this feature is implemented, it is critical to understand in which queue of the run loop each of these operations takes place.

Promises are resolved in the `actions` queue, the router transitions (and thus calling the model hook) take place in `routerTransitions`, and the DOM rendering happens in `render`. Let's see the queues again in descending order of their priority:

["sync", **"actions"**, **"routerTransitions"**, **"render"**, "afterRender", "destroy"]

When the router makes the call to the model hook of a route, the run loop is in `routerTransitions`. If a promise is returned, it is put into the `actions` queue. Now, the run loop needs to clear that queue next since it has a higher priority. This ensures that no tasks in the `render` queue are processed while any returned promise waits to resolve. In UI terms, nothing is painted on screen for that route until all data is returned by the promise.

The resulting user experience is that everything (the whole list of groups) appears at once instead of being rendered in steps. In some cases, like a dashboard that is comprised of many small widgets, data that "pops in" later is desirable, and is also possible.

### Acceptance tests that are clear and concise

Acceptance tests ensure that an application works properly by automating the user flow of its main features. They are a safety net against regression, the possibility that introducing a new feature breaks an old one (or several). Consequently, they have great value and allow developers to call it a day with a clear(er) conscience.

Oftentimes though, if acceptance tests are embarked upon at all, they are unfortunately abandoned because they can be fragile, time-consuming to maintain and run, or all of these.

The ember-testing package, which is an integral part of Ember, lowers the bar significantly by providing an extensible tool of test helpers and by making the tests run very quickly. Above that, the tests themselves are as clear as they can be so developers are less likely to shy away from reading and maintaining them.

The test for creating a group could look like this:

```javascript
test('Create a group', function() {
  visit('/groups');
  click('.new-group-button:contains("Create group")');
  fillIn('input[name="name"]', "Hack Hands");
  fillIn('input[name="url"]', "http://hackhands.com");
  click('button:contains("Save")');

  andThen(function() {
    equal($('.group-list .group').length, 1, "The new group appears in the list");
    equal($('.group').text(), "Hack Hands", "The name of the group is displayed");
    equal($('.group').text(), "http://hackhands.com", "The URL of the group is displayed");
  });
});

```

The first thing that comes to mind (at least to mine) when reading such a test is that it looks beautiful and is self-explanatory.

If you think about how user interaction with the browser works, you start to become curious about how such a test can work reliably. User events are processed asynchronously so there is no guarantee that the result of that event is already in the DOM before the next step in the test is processed. The same is true for `visit('/bands')`. Router transitions are asynchronous so if there was not any wizardry going on behind the scenes, the test would move on to the next line and try to click on the "New group" button which is not yet there since the route is still in mid-transition.

Of course there is some high-level wizardry happening behind the curtains. The above helpers provided by the ember-testing package (visit, click, fillIn, andThen) all [return a promise](https://github.com/emberjs/ember.js/blob/5a28224589b0eff8e4a7306a58c8c696272319e4/packages/ember-testing/lib/helpers.js#L167-203) that is resolved only after all asynchronous operations have been completed.

So that explains how the tests work at all but how is it that the next helper in the test waits for all async actions triggered by the preceding one to finish? Shouldn't we see the `then`-s, a tell-tale sign of working with promises? It should look something like this:

```javascript
test('Create a group', function() {
  visit('/groups')
  .then(function() {
    return click('.new-group-button:contains("Create group")');
  })
  .then(function() {
    return fillIn('input[name="name"]', "Hack Hands");
  })
  (...)
});

```

In fact, the binding between two subsequent lines is hidden by the testing framework. The key is a shared, global promise (called `Ember.Test.lastPromise`) that is [reassigned to the result of each helper's promise resolution](https://github.com/emberjs/ember.js/blob/5a28224589b0eff8e4a7306a58c8c696272319e4/packages/ember-testing/lib/test.js#L281-313). In practice that means that the subsequent calls (visit(...), click(...), fillIn(...), fillIn(...), etc.) are chained together and will orderly wait for their turn.

If you would like to see an even deeper analysis, check out [Cory Forsyth's blog post](http://coryforsyth.com/2014/07/10/demystifing-ember-async-testing/) on the subject.

### Communicating with the backend using Ember Data

Ember Data is a (and probably "the") model layer library for Ember.js. One among its numerous features is that it abstracts away communicating with the backend APIs. Just by defining your models and your adapter(s), the usual CRUD operations become one-liners. To wit, here is how to load all the groups:

```javascript
// app/routes/groups.js
export default Ember.Route.extend({
  model: function() {
    return this.store.find('group');
  }
});

```

Since we do not have the "privilege" of reloading the whole page to display a success message when a group is created, it is important to be able to react to the result of these operations.

Unsurprisingly, promises come to the rescue. Most methods in Ember Data that interface with the back-end return a promise and so the whole arsenal of the RSVP.js library is at our disposal.

To see an example, let's say we want to show a hooray message if the group creation is successful. If it fails, let the user see an "Oops!" blurb and have an error report sent to our developer team:

```javascript
// app/controllers/groups/new.js
export default Ember.Route.extend({
  (...)
  actions: {
    createGroup: function() {
      var controller = this,
             group = this.get('model');

      group.save().then(function() {
        controller.set('groupCreationResult', 'yay!'); // this renders the hooray message
        controller.transitionToRoute('group', group);
      }, function(error) {
        controller.set('groupCreationResult', 'nay'); // this shows the "Oops!"
        controller.send('wakeUpDevs', 'Group creation failed: ' + error);
      })
      ['catch'](function(error) {
        controller.send('wakeUpDevs', error);
      });
    }
  }
});

```

Since `group.save()` returns a promise, it is easy to add a fulfillment handler for the case when the save was successful, and a rejection handler for failure.
Adding a simple rejection handler at the end of the chain (see the `['catch']` above) is to prevent swallowing any errors and is a best practice with promises.

Loading records, reloading a single record, saving and deleting all return promises so you can choose to react to successes and handle errors in any way you see fit in these scenarios.

### Wrapping up

I showed several scenarios where Ember.js relies on promises to solve interesting problems in ingenious ways. Knowing how these concepts work under the hood not only makes you appreciate the finesse of the implementation, but it also enables you to leverage promises for the problems you come across. An example of this could be returning a composite promise from your model hook that fetches from several endpoints or trying to save a record several times before giving up.

There is a whole lot more to like and learn about Ember so if I piqued your interest, I suggest you [read my book](http://balinterdi.com/rock-and-roll-with-emberjs?utm_source=hackhands-ember-promises&utm_campaign=guest-blogging&utm_medium=referral), Rock and Roll with Ember.js. As its title suggests, you are in for quite a ride!

