![Ionic 2 + Angular 2](http://ionicframework.com/img/blog/ionic-angular-v2.jpg)

# Important update

Since Ionic 2.0-RC.0 the SqlStorage package that this tutorial rely on has been removed, and the WebSql fallback support has been dropped, you may be able to use versioning to only SqLite in mobile but web version has been sent to the trash, i'd suggest to stop thinking in Sql like databases and start learning NoSql since it seems browsers are moving into it (specially since WebSql specs are deprecated and the only browser that still works WebSql is Chrome).

## Introduction

The hybrid mobile app market has been expanding a lot since the release of `Ionic 2`, built with `Angular 2`. However, hybrid app structure standards are still very limited in Ionic 2.

This guide will cover building an essential part of the app: `databases`. This may not be a must-have in all applications, but if you choose the path of `offline first` for mobile app (usually an easy choice since a mobile device often gets disrupted signal when used without wi-fi), you'll have to rely on a local database to store data.

## Guide structure

This guide will teach you the following parts of database management in Ionic 2:

1. Setting up the guide environment.
2. Creating and opening the database.
3. Querying the database.
3. Using raw queries for getting insertions IDs.
4. Database versioning.
5. Backing up to `online` databases.
6. Resetting databases.

***

## Setting up the guide environment

The first thing to do is creating a new Ionic 2 app, if you are adding this feature to an existing app jump to the next section.

### Prerequisites

First we need to have `Node` and `NPM` (Node Package Manager) installed. Installation is relatively straightforward and covered thoroughly in other guides.

Next let's install Ionic 2 with npm, open a console and type the following commands which will install both `cordova` and `ionic@beta` which is the current version of ionic 2 since it's still in beta as of 24/06/2016.

```sh
npm i -g cordova ionic@beta
```

### Creating new app

After the Ionic CLI (Command Line Interface) is installed, we need to create a new app; we will use the side menu starter repo. Paste this in your terminal:

```sh
ionic start ionicDBManagerGuide sidemenu --v2 --ts && cd ionicDBManagerGuide && ionic serve
```

That will start an Ionic 2 `typescript` app using the side menu starter, and after it finishes it will open the folder and start the app, your browser will open and show you your new app.

You'll probably see a lot of output in the console from Ionic about downloading and installing modules and building your app before showing it in the browser. The whole process could take between one and five minutes, depending on your ethernet bandwidth and your computer's CPU, RAM, and drive speed. At the end your browser should show something similar to this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/136c91ef-0b9a-4b18-a39d-d2476d4096e5.png)

**Note 1:** In case you're not very skilled with the terminal, using `&&` between commands ensure all of them are run in serial way, and if one fails the next ones doesn't execute.

**Note 2:** Also note this command is made to work in unix-based terminals, so if you're in a different OS like `Windows`, you'll need to edit that command accordingly (or install a unix terminal).

***

## Creating and opening the database

Now here's where the fun starts. First let's talk a bit of theory. Angular 2 scopes isolate each component, which is great since errors in one component will not affect the other. Thus, using scopes, we can find and fix bugs more easily. However, this also means that if we want something to be available through the app, we need to use a provider (or a service, this days both are almost the same).

Next in the line is the database library selection. There are a few options when it comes to choosing a database library:

1. Local Storage (not good for critical or persistent data, can be erased by OS when running out of memory).
2. Cache Storage (better used with Service Workers, however that's an advanced use case and most likely not very useful in case of tabular data).
3. SQLite with WebSQL fallback (our choice and the defacto for many production ready hybrid apps).

We will be using `SQLite` for this purpose with `WebSql` fallback when in developing.

**Note:** We use the `WebSQL` fallback is to deploy the app both as a mobile app and as a web app. Sometimes the app isn't supported by the device so with this fallback, as good as the device have `Chrome` installed, they would be able to open the web version and use the app like that, also note i say `Chrome` because `Firefox` doesn't support `WebSQL`, and other browsers lack `internationalization api` support, thus you'll need to use polyfills to bridge the gap between browsers.

### Creating the database provider

`Ionic CLI` comes with a handy utility called a [generator](http://ionicframework.com/docs/v2/cli/generate/) that allows for quick prototyping.  To use it, list the available generators and then choose the one you want (note the shorthand of `generator` as `g`):

**Listing:**

```sh
ionic g --list
```

Which should output something like this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fc40b883-6892-4805-a038-41046b4f7794.png)

Next, we need a provider:

```sh
ionic g provider DBProvider
```

This generates the following folder structure in your app:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b02ca5fa-978f-47f3-955a-1d2bfb9aab3b.png)

Now let's delete the unnecessary code in there. After some paring, the code will look like this. The basic class we start from:

```ts
import { Injectable } from '@angular/core';

@Injectable()
export class DBProvider {
  constructor() {
  }
}
```

**Note:** Currently the generate command doesn't respect the casing and force kebab casing on the name, so please correct the class name as the above (`DBProvider` instead of `DbProvider`).

### Choosing and using database adapter

Now here's one of the first design decisions we'll make. We're going to use `SQLite` as the database technology and `WebSQL` as fallback in order to support databases during development. We will allow the app to be published as a web app as explained previously to support devices that Cordova won't. At the very least, our device needs to support Chrome.

But that's not the decision we need to make. Here's the deal, there's currently 2 ways to deal with `SQLite` databases in Ionic 2, one is through [Ionic native's SqLite adapter](http://ionicframework.com/docs/v2/native/sqlite/) and the other is through [Ionic's SqlStorage Api](http://ionicframework.com/docs/v2/api/platform/storage/SqlStorage). The difference between both is that the former has nice transaction methods bridged directly of [Cordova's SQLite plugin](http://ngcordova.com/docs/plugins/sqlite/), while the latter automatically configures `SQLStorage` in mobile and `WebSQL` in web browsers.

It's hard to deal with the `Ionic native` implementation since the function [openDatabase](https://github.com/driftyco/ionic-native/blob/master/src/plugins/sqlite.ts#L73-L83) only allows for the SQLite plugin and thus does not work when in dev. Any intent on getting the `Ionic native` staff into supporting the WebSQL fallback is useless, so we will use the `SQLStorage API` instead, though we will need to reach the transaction methods ourselves.

Right, now let's finally open the database, first enable `moduleResolution: node` in the `compilerOptions` to use barrels with just the index file by editing the `tscondfig.json` like this:

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "moduleResolution": "node", // Here's the new line
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true
  },
  "filesGlob": [
    "**/*.ts",
    "!node_modules/**/*"
  ],
  "exclude": [
    "node_modules",
    "typings/global",
    "typings/global.d.ts"
  ],
  "compileOnSave": false,
  "atom": {
    "rewriteTsconfig": false
  }
}

```

**Note:** This section is just to make the `barrel` so the imports are shorter.

Create the `app/providers/index.ts` file that will work as the barrel for all providers, take care of exporting first the contents of `CustomSQLite` as is a prerequisite of our `DBProvider`:

```ts
export * from './db-provider/db-provider.ts';
```

Let's add the `SQLStorage API` provider, first add the `Cordova SQLite plugin` to the project:

```sh
ionic plugin add cordova-sqlite-storage
```

Next instantiate the `SQLiteStorage` class; let's call it `storage` and add the `set` and `get` methods that will call the same methods from the `SqlStorage` class:

```ts
import { Injectable } from '@angular/core';
import { Storage, SqlStorage } from 'ionic-angular';

@Injectable()
export class DBProvider {
  private storage: any = new Storage(SqlStorage, {name: 'dbName'});
  constructor() {
  }

  get(key: string) {
    return this.storage.get(key);
  }

  getJson(key: string) {
    return new Promise((resolve, reject) => {
      this.storage.get(key).then((success) => {
        resolve(JSON.stringify(success));
      }, (err) => {
        reject(err);
      });
    });
  }

  set(key: string, value: any) {
    return this.storage.set(key, value);
  }

  setJson(key: string, value: any) {
    return this.storage.set(key, JSON.stringify(value));
  }

  remove(key: string) {
    return this.storage.remove(key);
  }
}
```

Note that we use type `any` for the `storage` property, we use `any` to be able to call the private methods of the `SqlStorage` class without typescript complaining about accessing a private property.

## Using the `DBProvider`

It's time to test if our new provider works.

### Providing the `DBProvider` to the app

If our test doesn't work, check previous steps to see whether you missed something or skipped a step. First add the provider to our `app/app.ts` component as a global provider in the `ionicBootstrap` method:

```ts
import { Component, ViewChild } from '@angular/core';
import { ionicBootstrap, Platform, Nav } from 'ionic-angular';
import { StatusBar } from 'ionic-native';

import { Page1 } from './pages/page1/page1';
import { Page2 } from './pages/page2/page2';

import { DBProvider } from './providers';

@Component({
  templateUrl: 'build/app.html'
})
class MyApp {

  ...
  constructor(private platform: Platform) {
    ...
  }
}

ionicBootstrap(MyApp, [
  DBProvider
]);
```

Next let's edit the first page by importing and adding the provider as a constructor dependency. We'll also set a variable in the database default storage, which uses the table `kv`. Last, we'll get and print the value obtained, and, just in case, let's log any error that occurs in the database query process:

```ts
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';

import { DBProvider } from '../../providers';

@Component({
  templateUrl: 'build/pages/page1/page1.html'
})
export class Page1 {

  constructor(
    private navCtrl: NavController,
    private db: DBProvider
  ) {
    db.set('dbVar', 'Database works')
    .then(() => db.get('dbVar'))
    .then((str) => console.log('Database string: ', str))
    .catch(err => console.error('DB error: ', err));
  }
}
```

In the current state, you should see the following in the browser in the `kv` table in your `WebSQL` database when testing with the command `ionic serve`:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6c25d6f9-f5db-403e-addc-a09316044a4c.png)

Also you should see something like this in your `Console`:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a1fa208c-2422-4905-946c-9f82f7a37d93.png)

If you see these things, it means your DBProvider works as expected.

### Creating and using basic query functions

The next few parts are feature-heavy, so most of this may be too advanced for a Ionic/Angular beginner. In case you feel like the material is too advanced, feel free to simply copy-paste the code of the functions I'm going to show you. For those who'd like to follow the process in-depth, I'll cover what I'm doing through comments above each function.

#### Query functions
First let's install `typings` package as global and add the `lodash` module as a dependency, then just use typing to install the lodash type definition:

```sh
npm i -g typings
npm i -S lodash
typings i dt~lodash -GS
```

Next let's add the query functions to our `DBProvider` class under the constructor:

```ts
import { map, flatMap, reduce, filter, split, trim, lowerCase } from 'lodash';

export class DBProvider {
  ...
  constructor() {
  }

  // Get the storage database directly, used in the transaction methods
  getDB() {
    return this.storage._strategy._db;
  }

  // Fetch 1 item from the database query result
  fetch (result: any) {
    if (result.rows.length <= 0) return false;
    return result.rows.item(0);
  }

  // Fetch all items from the database query result
  fetchAll (result: any) {
    let output = [];
    for (let i = 0; i < result.rows.length; i++) {
      output.push(result.rows.item(i));
    }
    return output;
  }

  // Single query, receives a SQL string and an array of args, also receive a 'raw' arg that determines
  // if it should return the query result directly:
  // query(sql: string, args: any[], raw = false) {
    // return this.db.query;
    return new Promise((resolve, reject) => {
      this.storage.query(sql, args).then((success) => {
        if (raw) {
          return resolve(success.res);
        }
        resolve(this.fetchAll(success.res));
      }, (err) => {
        reject(err);
      });
    });
  }

  // Helper transaction query
  queryTx(outTx: any, query: string, bindings: any[], raw: boolean) {
    return new Promise((resolve, reject) => {
      outTx.executeSql(query, bindings,
        (tx, success) => {
          if (raw) {
            return resolve(success);
          }
          resolve(this.fetchAll(success));
        },
        (tx, error) => {
          reject(error);
          return true;
        }
      );
    });
  }

  // Transaction query accepts an object whose query property is an array of query strings
  // and its args property is a 2D array of arguments for each query and maps each array in the
  // args array to a query string. The 'raw'' arg tells if it should return the query result directly.
  queryAll(SQL: any, raw: boolean = false) {
    return new Promise((resolve, reject) => {
      let promises = [];
      this.getDB().transaction((tx) => {
        for (let i = 0; i < SQL.query.length; i++) {
          promises.push(
            this.queryTx(tx, SQL.query[i], SQL.args[i], raw)
          );
        }
        return Promise.all(promises).then((success) => {
          resolve(success);
        }, (err) => {
          reject(err);
        });
      });
    });
  }
}
```

With this we are ready to make our first query, let's add a query to our `Page1` class constructor:

```ts
export class Page1 {

  constructor(
    private navCtrl: NavController,
    private db: DBProvider
  ) {
    db.set('dbVar', 'Database works')
    .then(() => db.get('dbVar'))
    .then(str => console.log('Database string: ', str))
    .then(() => db.query('SELECT * FROM kv WHERE key=?', ['dbVar']))
    .then(str => console.log('Query result: ', str))
    .catch(err => console.error('DB error: ', err));
  }
}
```

Here we're asking the database to get the value asociated with the `dbVar` string in the `kv` table in the database, next expanded output in browser console should look like this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ddc1ff34-b139-47fb-b17b-8d77eaa36ab9.png)

## Backup feature

The backup feature is meant to be used as a support mechanism. It will get all data from the tables you choose from the database, and serialize that data in a way so that we can send and receive from/to server and are able to back up and restore the database wherever we want.

Please note that this is mostly a serialization feature, which means you'll have to take care of sending and storing the string in server. This includes sending a very big string-ified `JSON`. Since databases get big fast, the serialized string gets larger big as the database grows, so, at some point, using a single HTTP post request will not be possible. Thus you will have to figure out how to send that string in chunks, through streaming or using sockets.

### Getting the schema from the table itself

Let's set the method that will allow us to know the structure of the database to backup, denominated `Schema`. This is an advanced feature that uses regex (Regular Expression) matching heavily:

```ts
export class DBProvider {
  ...
  queryAll(SQL: any, raw: boolean = false) { ... }

  // Helper function to get field name and type from the SQL query returned for each field from the 'sqlite_master' table
  getFieldFromSql(sql: string) {
    sql = trim(sql);
    let fieldArr = sql.split(/\s+/);
    return {
      name: fieldArr[0],
      type: fieldArr.slice(1).join(' '),
      sql: sql
    };
  }

  // Queries the 'sqlite_master' table, uses `regexp` to separate each field from each table, and returns a JSON object
  // with the name of the table, its fields and their names, types and separated SQLs.
  // itself used to create the field
  getSchema() {
    let SQL = {
      query: [
        `SELECT name, sql FROM sqlite_master
         WHERE type="table" AND name NOT IN
         ("sqlite_sequence","__WebKitDatabaseInfoTable__","kv") ORDER BY name`
      ],
      args: [[]]
    };
    return this.queryAll(SQL)
    .then((success) => {
      let splitFieldsPattern = /(?:\(.+?\))|(,)/ig;
      let schema = map(success[0], (table: any) => {
        let sql = table.sql.trim().match(/create table \w+\s?\(([\s\S]*)\)$/i)[1];
        let fieldMatch, matches = [];
        while (fieldMatch = splitFieldsPattern.exec(sql)) {
          if (fieldMatch[1] && fieldMatch[1].length === 1) {
            matches.push(fieldMatch);
          }
        }
        if (matches.length === 0) {
          return { name: table.name, fields: this.getFieldFromSql(sql) };
        }
        let fields = reduce(matches, (fieldArr, match, i) => {
          let prevIndex = i > 0 ? matches[i - 1].index + 1 : 0;
          let fieldSql = sql.slice(prevIndex, match.index);
          fieldArr.push(this.getFieldFromSql(fieldSql));
          if (i === matches.length - 1) {
            fieldSql = sql.slice(match.index + 1);
            let field = this.getFieldFromSql(fieldSql);
            if (!lowerCase(field.name).includes('constraint')) {
              fieldArr.push(field);
            }
          }
          return fieldArr;
        }, []);
        return { name: table.name, fields: fields };
      });
      return schema;
    });
  }
```

## DB versioning feature

WIP
