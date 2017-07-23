## Package Managers

Ruby has [Bundler](http://bundler.io/).

PHP has [Composer](https://getcomposer.org/). 

Rust has [Cargo](https://crates.io/).

Python has [pip](https://pip.pypa.io/en/stable/).

These package managers automate the process of installing and managing libraries and dependencies.

In Node.js, we have [npm](https://www.npmjs.com/) as the default package manager, which is automatically included when Node.js is installed.

Npm is not perfect, and there are a number of open source alternatives created to solve some of its issues like [ied](http://gugel.io/ied/), [pnpm](http://ricostacruz.com/pnpm/), and more recently, [yarn](https://yarnpkg.com)

### Introducing Yarn

**Yarn** was released by Facebook in collaboration with Exponent, Google, and Tilde in October 2016.

There's a post on [Facebook Code](https://code.facebook.com/posts/1840075619545360) that describe the reasons for building this new package manager.

Here are some highlights:

> ... as we scaled internally, we faced problems with consistency when installing dependencies across different machines and users, the amount of time it took to pull dependencies in, and had some security concerns with the way the npm client executes code from some of those dependencies automatically.

> We also had to work around issues with npm's shrinkwrap feature, which we used to lock down dependency versions.

> ... updating a single dependency with npm also updates many unrelated ones based on semantic versioning rules.

Many advantages of using Yarn aren't common knowledge yet. This guide will explore the advantages of Yarn over npm and will cover some basic commands that will help new users settle into using Yarn regularly.

Here's [Yarn's Github page](https://github.com/yarnpkg/yarn). It's a pretty popular project; at the time of this writing, it has 21,500 stars and more than 500 open issues. Yarn is compatible with the npm registry and has the same set of features as npm, but it operates faster and in a more reliable way. Let's take a look.

# Installation

You can install yarn with npm but this is *not recommended*:
```
npm install -g yarn
```

Instead, if you're on Windows, download the [installer](https://yarnpkg.com/latest.msi) (you need to have Node.js installed).

If you're using iOS or a Unix environment, the easiest way is via a shell script:
```
curl -o- -L https://yarnpkg.com/install.sh | bash
```

In [this page](https://yarnpkg.com/en/docs/install) you can find more information and other installation methods.

# How it works

As we know, Node.js dependencies are placed in the `node_modules` directory of your project. 

These dependencies are installed in a non-deterministically way, which means that the directory structure and dependency tree depend on the order dependencies are installed. Such a dependency is problematic because it can differ from machine to machine.

To solve this issue, Yarn uses a lockfile (`yarn.lock`) to ensure that everyone has the same version of every single file, resulting in the exact same file structure of the `node_modules` directory across all machines.

When dependencies are resolved, Yarn first looks in a local (global) cache to see if the package has been downloaded already. This, with Yarn's ability to efficiently queue up and parallelize requests, makes Yarn faster than npm.

So let's test it.

# NPM vs Yarn

The comparisons outlined here were made using npm 4.05 and Yarn 0.18.1.

To initialize a project with npm we use `npm init`:

![npm init](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd47560b-2ed1-4623-9d94-c8c556b6dd00.png)

Yarn has the same `init` command, but with a slightly different set of questions and answers:

![yarn init](https://raw.githubusercontent.com/pluralsight/guides/master/images/065a7752-b258-4eb5-ad37-8390b71a4217.png)


To install a dependency and save it to `package.json`, for example [express](http://expressjs.com/) (which has more than twenty dependencies), in npm we execute:
```
npm install --save express
```

Using Yarn:
```
yarn add express
```

One little advantage of using Yarn is that you have to type less. Well, as long as you're not using aliases like `npm i -S express`.

But let's time the execution of these commands.

Npm:

![time npm install](https://raw.githubusercontent.com/pluralsight/guides/master/images/c966fadf-35e2-482b-8190-883574c5973d.gif)


Yarn:

![time yarn add](https://raw.githubusercontent.com/pluralsight/guides/master/images/b7f0d552-7d8e-46cc-a0e7-beafec92c1e3.gif)


As you can see from the images, npm took 3.120 seconds, while Yarn took 2.588 seconds.

You can also see that the output of Yarn is more descriptive and prettier. The emojis are only available on Mac though.

Now, let's delete the `node_modules` directory and repeat the test to see Yarn's caching in action:

![yarn cache usage](https://raw.githubusercontent.com/pluralsight/guides/master/images/8e82a107-a6f8-4178-bf2c-303ab44b5d36.gif)


This time, Yarn took 1.455 seconds, an improvement of 44% over the previous execution. And 53% over npm.

Of course, you may get different results, but **Yarn will beat npm every time.**

We can also see that a `yarn.lock` file is generated (some parts are omitted for shortness):
```
# THIS IS AN AUTOGENERATED FILE. DO NOT EDIT THIS FILE DIRECTLY.
# yarn lockfile v1


accepts@~1.3.3:
  version "1.3.3"
  resolved "https://registry.yarnpkg.com/accepts/-/accepts-1.3.3.tgz#c3ca7434938648c3e0d9c1e328dd68b622c284ca"
  dependencies:
    mime-types "~2.1.11"
    negotiator "0.6.1"

array-flatten@1.1.1:
  version "1.1.1"
  resolved "https://registry.yarnpkg.com/array-flatten/-/array-flatten-1.1.1.tgz#9a5f699051b1e7073328f2a008968b64ea2955d2"

content-disposition@0.5.1:
  version "0.5.1"
  resolved "https://registry.yarnpkg.com/content-disposition/-/content-disposition-0.5.1.tgz#87476c6a67c8daa87e32e87616df883ba7fb071b"

content-type@~1.0.2:
  version "1.0.2"
  resolved "https://registry.yarnpkg.com/content-type/-/content-type-1.0.2.tgz#b7d113aee7a8dd27bd21133c4dc2529df1721eed"

cookie-signature@1.0.6:
  version "1.0.6"
  resolved "https://registry.yarnpkg.com/cookie-signature/-/cookie-signature-1.0.6.tgz#e303a882b342cc3ee8ca513a79999734dab3ae2c"

cookie@0.3.1:
  version "0.3.1"
  resolved "https://registry.yarnpkg.com/cookie/-/cookie-0.3.1.tgz#e7e0a1f9ef43b4c8ba925c5c5a96e806d16873bb"

debug@~2.2.0:
  version "2.2.0"
  resolved "https://registry.yarnpkg.com/debug/-/debug-2.2.0.tgz#f87057e995b1a1f6ae6a4960664137bc56f039da"
  dependencies:
    ms "0.7.1"

depd@~1.1.0:
  version "1.1.0"
  resolved "https://registry.yarnpkg.com/depd/-/depd-1.1.0.tgz#e1bd82c6aab6ced965b97b88b17ed3e528ca18c3"

destroy@~1.0.4:
  version "1.0.4"
  resolved "https://registry.yarnpkg.com/destroy/-/destroy-1.0.4.tgz#978857442c44749e4206613e37946205826abd80"

ee-first@1.1.1:
  version "1.1.1"
  resolved "https://registry.yarnpkg.com/ee-first/-/ee-first-1.1.1.tgz#590c61156b0ae2f4f0255732a158b266bc56b21d"

encodeurl@~1.0.1:
  version "1.0.1"
  resolved "https://registry.yarnpkg.com/encodeurl/-/encodeurl-1.0.1.tgz#79e3d58655346909fe6f0f45a5de68103b294d20"

escape-html@~1.0.3:
  version "1.0.3"
  resolved "https://registry.yarnpkg.com/escape-html/-/escape-html-1.0.3.tgz#0258eae4d3d0c0974de1c169188ef0051d1d1988"

etag@~1.7.0:
  version "1.7.0"
  resolved "https://registry.yarnpkg.com/etag/-/etag-1.7.0.tgz#03d30b5f67dd6e632d2945d30d6652731a34d5d8"

express@^4.14.0:
  version "4.14.0"
  resolved "https://registry.yarnpkg.com/express/-/express-4.14.0.tgz#c1ee3f42cdc891fb3dc650a8922d51ec847d0d66"
  dependencies:
    accepts "~1.3.3"
    array-flatten "1.1.1"
    content-disposition "0.5.1"
    content-type "~1.0.2"
    cookie "0.3.1"
    cookie-signature "1.0.6"
    debug "~2.2.0"
    depd "~1.1.0"
    encodeurl "~1.0.1"
    escape-html "~1.0.3"
    etag "~1.7.0"
    finalhandler "0.5.0"
    fresh "0.3.0"
    merge-descriptors "1.0.1"
    methods "~1.1.2"
    on-finished "~2.3.0"
    parseurl "~1.3.1"
    path-to-regexp "0.1.7"
    proxy-addr "~1.1.2"
    qs "6.2.0"
    range-parser "~1.2.0"
    send "0.14.1"
    serve-static "~1.11.1"
    type-is "~1.6.13"
    utils-merge "1.0.0"
    vary "~1.1.0"

...

vary@~1.1.0:
  version "1.1.0"
  resolved "https://registry.yarnpkg.com/vary/-/vary-1.1.0.tgz#e1e5affbbd16ae768dd2674394b9ad3022653140"
```

Notice that Yarn uses `https://registry.yarnpkg.com` instead of `https://registry.npmjs.org`. 

In a [podcast episode of NodeUp](http://nodeup.com/onehundrednine) (around the 40:30 mark) [James Kyle](https://github.com/thejameskyle) (one of the main developers of Yarn) said that, for now, `registry.yarnpkg.com` is just a CNAME record that goes to the [Cloudflare](https://www.cloudflare.com/) network and redirects to the NPM registry for experimental purposes, and to add some performance features.

Now compare it to the file `npm-shrinkwrap.json` (which serves a similar purpose than `yarn.lock`), generated when `npm shrinkwrap` is executed (some parts are omitted for shortness):
```
{
  "name": "npm",
  "version": "1.0.0",
  "dependencies": {
    "accepts": {
      "version": "1.3.3",
      "from": "accepts@>=1.3.3 <1.4.0",
      "resolved": "https://registry.npmjs.org/accepts/-/accepts-1.3.3.tgz"
    },
    "array-flatten": {
      "version": "1.1.1",
      "from": "array-flatten@1.1.1",
      "resolved": "https://registry.npmjs.org/array-flatten/-/array-flatten-1.1.1.tgz"
    },
    "content-disposition": {
      "version": "0.5.1",
      "from": "content-disposition@0.5.1",
      "resolved": "https://registry.npmjs.org/content-disposition/-/content-disposition-0.5.1.tgz"
    },
    "content-type": {
      "version": "1.0.2",
      "from": "content-type@>=1.0.2 <1.1.0",
      "resolved": "https://registry.npmjs.org/content-type/-/content-type-1.0.2.tgz"
    },
    "cookie": {
      "version": "0.3.1",
      "from": "cookie@0.3.1",
      "resolved": "https://registry.npmjs.org/cookie/-/cookie-0.3.1.tgz"
    },
    "cookie-signature": {
      "version": "1.0.6",
      "from": "cookie-signature@1.0.6",
      "resolved": "https://registry.npmjs.org/cookie-signature/-/cookie-signature-1.0.6.tgz"
    },
    "debug": {
      "version": "2.2.0",
      "from": "debug@>=2.2.0 <2.3.0",
      "resolved": "https://registry.npmjs.org/debug/-/debug-2.2.0.tgz"
    },
    "depd": {
      "version": "1.1.0",
      "from": "depd@>=1.1.0 <1.2.0",
      "resolved": "https://registry.npmjs.org/depd/-/depd-1.1.0.tgz"
    },
    "destroy": {
      "version": "1.0.4",
      "from": "destroy@>=1.0.4 <1.1.0",
      "resolved": "https://registry.npmjs.org/destroy/-/destroy-1.0.4.tgz"
    },
    "ee-first": {
      "version": "1.1.1",
      "from": "ee-first@1.1.1",
      "resolved": "https://registry.npmjs.org/ee-first/-/ee-first-1.1.1.tgz"
    },
    "encodeurl": {
      "version": "1.0.1",
      "from": "encodeurl@>=1.0.1 <1.1.0",
      "resolved": "https://registry.npmjs.org/encodeurl/-/encodeurl-1.0.1.tgz"
    },
    "escape-html": {
      "version": "1.0.3",
      "from": "escape-html@>=1.0.3 <1.1.0",
      "resolved": "https://registry.npmjs.org/escape-html/-/escape-html-1.0.3.tgz"
    },
    "etag": {
      "version": "1.7.0",
      "from": "etag@>=1.7.0 <1.8.0",
      "resolved": "https://registry.npmjs.org/etag/-/etag-1.7.0.tgz"
    },
    "express": {
      "version": "4.14.0",
      "from": "express@>=4.14.0 <5.0.0",
      "resolved": "https://registry.npmjs.org/express/-/express-4.14.0.tgz"
    },
	
	...
	
    "vary": {
      "version": "1.1.0",
      "from": "vary@>=1.1.0 <1.2.0",
      "resolved": "https://registry.npmjs.org/vary/-/vary-1.1.0.tgz"
    }
  }
}
```

The advantage of using Yarn is that `yarn.lock` is generated automatically; with npm you have to execute `npm shrinkwrap` manually.

One important thing, `yarn.lock` [should always be added to source control](https://yarnpkg.com/blog/2016/11/24/lockfiles-for-all). Remember that this file ensures the deterministic installation of the dependencies.

Now, if you specify a package version, you'll notice an important difference in the generated `package.json` file.

For example, if you execute:
```
npm install --save express@4.13.4 
```

It will generate:
```
{
  ...
  "dependencies": {
    "express": "^4.13.4"
  }
}
```

Executing:
```
yarn add express@4.13.4 
```

Will generate:
```
{
  ...
  "dependencies": {
    "express": "4.13.4"
  }
}
```

Did you notice the missing caret(`^`) in Yarn's `package.json` file?

Many people recommended **not** using carets in our `package.json` files.

The caret will update the dependency to the most recent minor version when it's installed. For example, if there's a `2.4.1` version available, this will be installed even if `^2.3.1` is the one specified. This can cause trouble in some cases because sometimes breaking changes are introduced even in minor releases.

If you want to specify an exact version rather than using npm's semantic version range operator (the caret), you have to use the `--save-exact` (or its alias `-E`), as in:
```
npm install --save-exact express@4.13.4 
```

However, that's something you have to know and keep in the back of your head. On the other hand, Yarn does this "caret-handling" for you. If this is something you want, it depends of your preferences and the libraries you're using.

Moving forward, if you already have a `package.json` file, you can install the dependencies with:
```
yarn install
```

Or just:
```
yarn
```

If you want to remove a dependency and update the `package.json` file to reflect that, with npm you need to execute:
```
npm uninstall --save express
```

With Yarn, you need to execute:
```
yarn remove express
```

That will update your `package.json` and `yarn.lock` files by default.

The command `yarn upgrade [package]` will upgrade all the packages (or a single named package) to their latest version ([some rules apply](https://yarnpkg.com/en/docs/cli/upgrade)), and using `yarn upgrade package@version` will upgrade (or downgrade) an installed package to the specified version. In all cases, the `yarn.lock` file will be recreated as well.

Next I'll cover a key difference between npm and Yarn.

## npm dedupe

One feature missing on Yarn is an equivalent of [npm dedupe](https://docs.npmjs.com/cli/dedupe), a command to reduce duplicated dependencies. Since a project can have many versions of the same dependency, this `dedupe` command can come in handy.

For background, `npm dedupe` searches the dependency tree and attempts to simplify the overall directory structure by moving dependencies further up the tree (even if duplicates are not found), where they can be more effectively shared by multiple dependent packages. If a suitable version already exists at the target location, it will be left untouched, but the other duplicates will be deleted. This will result in both a flat and deduplicated tree.

On the other hand, the Yarn `install` command has a `--flat` flag:
```
yarn install --flat
```

On the first run, it will prompt you to choose a *single version* for each package that has more than one version on the dependency tree. These will be added to your `package.json` under a `resolutions` field. **But this is not the same as `npm dedupe`, which removes duplicates by deletion.**

This difference means that the `node_modules` directory generated by Yarn could take more space on disk than the one generated by npm.

However, this tradeoff might not impact you, your workflow, or your business significantly. 

## Other helpful commands in Yarn

This section covers other Yarn commands that you should know. You can see the complete list of Yarn commands and their options [here](https://yarnpkg.com/en/docs/cli/).

For one, Yarn comes with a [license checker](https://yarnpkg.com/en/docs/cli/licenses). It will give you the license (and URL to the source code) associated with each package:
```
yarn licenses ls
```
![yarn licenses](https://raw.githubusercontent.com/pluralsight/guides/master/images/db8459ae-9bd2-4680-8888-8e52450a1e3d.gif)

In addition to `ls`, `yarn licenses generate-disclaimer` will output the content of all licenses from all the packages you have installed.

To fetch information about a package, use [yarn info](https://yarnpkg.com/en/docs/cli/info):
```
yarn info cookie
```
![yarn info](https://raw.githubusercontent.com/pluralsight/guides/master/images/a205ec05-b989-4128-8843-a663b632687d.gif)


You can also ask for the version information, such as the currently installed version, the desired version based on semver, and the latest available version, of one or more packages of your `packages.json` file:
```
yarn outdated
```
![yarn outdated](https://raw.githubusercontent.com/pluralsight/guides/master/images/ac21f2d3-69fb-4b98-a47d-c62dd87e5276.png)


Finally, Yarn can show information about why a package, package folder, or file within a package folder is installed:
```
yarn why xxx
```
![yarn why](https://raw.githubusercontent.com/pluralsight/guides/master/images/b8b6e859-208a-492b-b628-8a627339fcea.png)

# Conclusion

Yarn is already used in production at Facebook. Furthermore, as demonstrated in this guide, it's a great high-performance alternative to npm. 

For simple personal projects, perhaps the one you use doesn't make much of a difference. However, for bigger projects (and depending on your needs), Yarn might provide a clear advantage over the alternatives.

___

Let me know in the comments if you have any information to add. If any part of this guide is incorrect, please open a pull request on Github! Thanks for reading.
