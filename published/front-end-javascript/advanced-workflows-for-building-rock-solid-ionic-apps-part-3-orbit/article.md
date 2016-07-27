![Advanced workflows for building rock-solid Ionic apps. Part 3: Orbit](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/blog-03.png)

The air is getting a little thin in App-Development-Orbit! Luckily Part 3 of this series about developing Ionic apps with **Generator-M-Ionic** comes with a sweet oxygen mask full of life-saving elements: environments, CORS proxies and build tools for app icons, splash screens, build variables, and app delivery. Take a deep breath before we extend the project we made space-ready in [part 2](http://tutorials.pluralsight.com/front-end-javascript/advanced-workflows-for-building-rock-solid-ionic-apps-part-2-mountain).

### Gulp environments
So you have your app all set up and have added plugins and other components while implementing your app logic. At some point you will probably want to exchange data with an external backend.
No problem. Just inject the good old Angular [$http service](https://docs.angularjs.org/api/ng/service/$http), handle all the HTTP calls with that nice Angular [promise API](https://docs.angularjs.org/api/ng/service/$q) and you're done. 

For instance, let's suppose I want to shoot some requests at the [Postman Echo API](https://echo.getpostman.com).
```js
$http.get('https://echo.getpostman.com/get') //$http service
.then(function (response) {
  console.log(response); //promise setup
});
```
Yeah! As simple as that!

> What if the customer changes the URL of the endpoint (that's the beta version of the API) to `https://blabber.getpostman.com`.

*"OK, if that keep's happening I'm going to put the URL in a constant"* you might think. Clever! Luckily your project already comes with a `Config` constant in `app/main/constants/config-const.js`. Just adapt your code.

```js
.constant('Config', {
  API_ENDPOINT: 'https://blabber.getpostman.com'
  // ..
});
```
```js
$http.get(Config.API_ENDPOINT + '/get')
.then(function (response) {
  console.log(response);
});
```
Genius!

> The customer tells you that some features you need to implement only work with the beta API `https://blabber.getpostman.com` and some only work with the production API `https://echo.getpostman.com`.

*"This will be annoying! I will need to exchange the endpoint URL all the time!"*

Worry not! [Environments](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/environments.md) to the rescue! 

There's two JSON files in `app/main/constants/` which you will fill with the endpoints:
```js
// env-dev.json
{
  "API_ENDPOINT": "https://blabber.getpostman.com"
}
```
```js
// env-prod.json
{
  "API_ENDPOINT": "https://echo.getpostman.com"
}
```
Then modify your controller code:
```js
$http.get(Config.ENV.API_ENDPOINT + '/get')
.then(function (response) {
  console.log(response);
});
```
Now if you run one of these:
```sh
gulp watch
# or
gulp build
# or
gulp --cordova "run <platform>"
# or anything similar
```
Without any flags it will default to the dev environment and your HTTP calls will end up against the `API_ENDPOINT` of your `env-dev.json`:  `https://blabber.getpostman.com`. How? **Because the environments task injects the contents of the JSON file in the `ENV` object of your `Config` constant**:
```js
angular.module('main')
.constant('Config', {

  // gulp environment: injects environment vars
  ENV: {
    /*inject-env*/
    'API_ENDPOINT': 'https://blabber.getpostman.com'
    /*endinject*/
  },
  // ..
});
```
On the other hand adding the `--env=prod` flag like below will send your HTTP calls to `https://echo.getpostman.com`, the `API_ENDPOINT` of your `env-prod.json`.
```
gulp watch --env=prod
```
Cool, huh?

Environments come in handy for a variety of tasks: managing different API keys, tokens, logging levels, feature switches and, as we just found out, API endpoints. The best part is that you can have as many environments as you want! A more detailed explanation of this incomprehensible-yet-so-appealing piece of magic is available in our **[Environments Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/environments.md)**.


### CORS & Proxying
Took care of that API-spaghetti. But wait what's that? Suddenly something like this shows up in your development console as you send an HTTP request. And it is bright red!
```
XMLHttpRequest cannot load https://echo.getpostman.com/. The
'Access-Control-Allow-Origin' header contains the invalid value ''.
Origin 'http://localhost:3000' is therefore not allowed access.
```
*"Hello, improperly configured REST API!"* Believe me, my team and I have been beaten by issues with [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) (Cross-Origin Resource Sharing) MANY times. 

So I created a little guide on how to deal with [CORS issues & proxying](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/cors_proxy.md). If you're dealing with CORS or proxying, it's a great read! There's a variety of options and depending in your project some might work better than others.

One of the most straight forward strategy, which is covered in my guide, is the [built-in proxy](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/cors_proxy.md#built-in-proxy) that comes with your Generator-M-Ionic project.

Remember that piece of code from earlier? This will actually run into a CORS error using that API endpoint:
```js
$http.get('https://echo.getpostman.com/get')
.then(function (response) {
  console.log(response);
});
```
In order to configure the built-in proxy, I'll just start `gulp watch` with the following parameters:
```sh
gulp watch --proxyPath=/proxy --proxyMapTo=https://echo.getpostman.com
```

And now the following request to my proxy will give me the response from `https://echo.getpostman.com/get` that I want.
```js
$http.get('/proxy/get')
.then(function (response) {
  console.log(response);
});
```
In fact, now I can send any request of the pattern `/proxy/**/*` and it'll be mapped to `https://echo.getpostman.com/**/*` through the proxy. This fits nicely with the generator's [Environments feature](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/environments.md):

```js
$http.get(Config.ENV.API_ENDPOINT + '/get')
.then(function (response) {
  console.log(response);
});
```
With the contents of your environment JSON files being:
```js
// env-dev.json
{
  "API_ENDPOINT": "/proxy"
}
```
```js
// env-prod.json
{
  "API_ENDPOINT": "https://echo.getpostman.com"
}
```
And then switch between firing against the proxy or directly at the API using the `--env` flag:
```sh
# will default to --env=dev and thus use the proxy
gulp watch --proxyPath=/proxy --proxyMapTo=https://echo.getpostman.com

# the built version will fire directly against the API
gulp build --env=prod
```
As simple as that! No more manual, tedious, error-prone changing of the endpoints.

Quite useful, isn't it?


### Default tasks
Well just typing that whole thing also is kind of tedious:
```sh
gulp watch --proxyPath=/proxy --proxyMapTo=https://echo.getpostman.com
```
Especially if you add `--env=prod` or `--no-open` to the list. The latter I like to use to prevent the browser from opening a new window every time. Wouldn't it be nice if my project just *knew* that I want those flags to run with `gulp watch`? We thought so too! That's why we built this neat little task called `gulp defaults`. 

The syntax is fairly easy:
```sh
gulp defaults --set="watch --no-open --proxyPath=/proxy --proxyMapTo=https://echo.getpostman.com"
```
Now every time I run `gulp watch`, Gulp will add those three flags for me. Yes! It even reminds me which flags I've set on every run, so I don't get confused.

![gulp defaults task](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/gulp_defaults.png)

Neat! Learn more about Gulp defaults, how to clear, overwrite and share defaults across your team via Git in our [Gulp defaults guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/gulp_defaults.md).

### Building and delivering your app
The first testable version of your app is ready. The customer has the following requirements for the hand-over:
- deliver two apps, one for each of the APIs
- to not confuse the customer, you need a different app icon and splash screen for each version
- show the date and time of the build in the app

Good thing you're using [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), which has nice tools for all of that!

#### Building your app
Building your app with Cordova is just as easy as running your app on a device:
```sh
gulp --cordova "build ios"
```
Remember that this implicitly runs `gulp build` first and then triggers Cordova's [build command](https://cordova.apache.org/docs/en/latest/reference/cordova-cli/index.html#cordova-build-command), as explained in our [Development Introduction](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/development_intro.md#using-the-cordova-cli). This command above is equivalent to the following statements:
```sh
gulp build
gulp --cordova "build ios" --no-build
```
However, for proper code-signing, it sometimes is necessary to go with Cordova's [prepare command](https://cordova.apache.org/docs/en/latest/reference/cordova-cli/index.html#cordova-prepare-command). Then open the platform's project located in `platforms/` and perform the code-signing, building, and all that. For instance, in [Xcode](https://developer.apple.com/xcode/), when building for iOS:
```sh
gulp --cordova "prepare ios" # also runs `gulp build` first
```

#### Build for different APIs
Since we've already configured our [environments](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/environments.md) properly, building for two different APIs is just as easy as developing for two different APIs. And we already know how to do that using `gulp watch` and the `--env` flag. Luckily, building an app for each environment is just as easy:
```sh
# build with dev environment
gulp --cordova "build ios" --env=dev

# build with prod environment
gulp --cordova "build ios" --env=prod
```
The `--env=dev` flag is not necessary, since the environment always defaults to the development environment with every command. However, I like to be explicit here to make the code more readable.

Ok, done with building apps for different APIs! Next challenge!

#### Build with different app icons and splash screens
I'm not going to go into the particulars of this one, since there's a perfectly nice [Icons and splash screens guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/icons_splash_screens.md) in our documentation. Whether you just have one set of icons and splash screens or ten, it's fairly simple to set them up and use them.

After proper configuration, all you need to do to build with different resources is add the `--res` flag. Check this out. 
```sh
# build with dev set of resources
gulp --cordova "build ios" --res=dev

# build with prod set of resources
gulp --cordova "build ios" --res=prod
```

Our code now builds with different icons and splash screens, as defined in the dev and prod environments. 

Next, how do we display the date and time of the build in our app?

#### Injecting variables from the command line
At some point, especially when you're automating the build process of your apps by [continuously integrating](https://en.wikipedia.org/wiki/Continuous_integration) with [Jenkins](https://jenkins.io/) or [Travis](https://travis-ci.org/), you might want to __inject data__ into your app through the command line. 

Granted, showing the date and time of a build in the app is not a very useful use-case. In reality, you will probably find yourself wanting to append build numbers into your app's version number, which in turn is read from the `config.xml`, and inject both build numbers and version number into the app. 

Guess what? There's a [guide for doing both of that](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/programmatically_change_configxml.md). But for now to keep things simple, we'll just stick with injecting a date.

You may have noticed that in your `Config` constant is another block that looks similar to the one where the  [environment variables](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/environments.md) get injected. That block is the place where you can inject variables into your app from the command line using `gulp watch` and inject any tasks that use `gulp build` implicitly. See how:

```sh
# running $(date +%s) will only work on Unix
gulp watch --buildVars="buildTime:$(date +%s),buildNo:12"
```
Running the above command will produce this output in the `Config` const:
```js
angular.module('main')
.constant('Config', {

  // gulp environment: injects environment vars
  ENV: {
    /*inject-env*/
    'API_ENDPOINT': 'https://blabber.getpostman.com'
    /*endinject*/
  },

  // gulp build-vars: injects build vars
  BUILD: {
    /*inject-build*/
    'buildTime': '1462972664',
    'buildNo': '12'
    /*endinject*/
  }

});
```
Now, you can map in one of your controllers:
```js
this.build = Config.BUILD;
```
Using the Angular [date filter](https://docs.angularjs.org/api/ng/filter/date) in the matching template, show the date and build number in your app:
```html
<ion-item class="item item-icon">
  <!-- multiply by 1000 to convert from unix to JS timestamp -->
  {{ctrl.build.buildTime * 1000 | date:'medium'}}
</ion-item>
<ion-item class="item item-icon">
  {{ctrl.build.buildNo}}
</ion-item>
```

#### Putting it all together
The only thing left to do is to merge whatever we just did for our build: build the app with different environments and different sets of resources, and inject variables from the command line:
```sh
# build dev version
gulp build --env=dev --res=dev --buildVars="buildTime:$(date +%s),buildNo:12"
gulp --cordova "build ios" --no-build

# build prod version
gulp build --env=prod --res=prod --buildVars="buildTime:$(date +%s),buildNo:12"
gulp --cordova "build ios" --no-build
```
At this point I like to explicitly separate the web app build from the Cordova build, so it doesn't get messy and it's a little more apparent what we are actually doing. __Do not forget the `--no-build` option when running the Cordova build__ or it will implicitly run `gulp build` and overwrite your previous `gulp build` without any of the options you supplied earlier.

### Deliver
Now that you have built two runnable apps, how do you get them to your customer for testing purposes?

- You can use the [Ionic Platform](http://ionic.io/platform) features once you've integrated them into your project using the [Ionic CLI](http://ionicframework.com/docs/cli/). See how, in our [Ionic Platform Integration Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/ecosystems/ionic_platform.md)
- Use our [Relution Enterprise Mobility Management](https://www.relution.io/en/) platform
- Use [Testflight](https://developer.apple.com/testflight/) and [HockeyApp](https://www.hockeyapp.net/features/), which are popular alternatives

Pick the one that suits you best, and get that customer feedback!


### Bang!
**\*confettiandglitter rainingdownonyou***

You've done it! Big and warm-hearted congratulations!

You fought yourself all the way through adventure-island's toddler-playground, grabbed your development pick-axe and dragged yourself on top of that mountain, only to jump in that space suit and explore app-orbit! You learned how to set up a project with [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), learned about its file structure, Git, quality assurance with linting & testing, sub-generators, Sass, Plugins, a lot of Gulp tasks, ecosystems, environments, CORS, proxies, build vars, and many many other things.

Stay curious, there's many more things to explore and don't forget to firmly pat yourself on the back! You have earned it!

### Sparkly future
We're living in exiting times; Angular 2 and Ionic 2 are around the corner, and Typescript and ES6 are becoming viable choices. We're on it! Would you like to see a version of [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic) with those technologies? What features would you like with that? Let us know!

### Get in touch
Feedback, ideas, comments regarding this blog post or any of the features discussed here are very welcome in either in the comments section below, at our [Generator-M-Ionic's Github repository](https://github.com/mwaylabs/generator-m-ionic), or the [Generator-M-Ionic Gitter Chat](https://gitter.im/mwaylabs/generator-m-ionic).

### Credits
Author: [Jonathan Grupp](https://github.com/gruppjo)  
Headline illustrations: [Christian Kahl](http://www.art-noir.net/)  
Special thanks to Volker Hahn, Mathias Maier & Tim Lancina

> Originally published July 7, 2016 on the [Ionic Blog](http://blog.ionic.io/advanced-workflows-for-building-rock-solid-ionic-apps-part-3/) in a slightly modified version.
