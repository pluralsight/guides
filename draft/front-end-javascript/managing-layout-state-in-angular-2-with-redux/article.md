# The benefits of controlling your application layout with Redux
 of the application is going to be a representation of the states of all elements in the layout - accordions, sidebars, pagination and everything that is pertinent to the use cases in the application. Reduxifying the layout
# Setup
 The examples  below will be done with [ng-boostrap]() since it's one of the most popular libraries with Angular 2 components for Bootstrap 4. However, you can also implement these examples with other component libraries such as [Material Design](https://github.com/angular/material2) by following the same design principles and making small adjustments to the code so that it works with the API of the corresponding library.
 
### Dependencies
 Here's what packages you need to install in order to start working:
 
**Redux**
 - [@ngrx/store + @ngrx/core](https://github.com/ngrx/store)
 - [@ngrx/effects](https://github.com/ngrx/effects)
 - [reselect](https://github.com/reactjs/reselect)
 - [ngrx-store-logger](https://github.com/btroncone/ngrx-store-logger)
 
** Bootstrap **
 - [Boostrap 4](https://github.com/twbs/bootstrap)
 - [ng-boostrap](https://github.com/ng-bootstrap/ng-bootstrap)
 
### Installing

To ensure a smooth setup, [Angular CLI](https://github.com/angular/angular-cli) will be used to initialize the applicaiton architecture. Make sure you have it installed globally before you proceed. In your terminal, write the following commands to initialize your Angular 2 app:

```
$ ng new redux-layout-tutorial-app
$ cd new redux-layout-tutorial-app
```


```
$ yarn add bootstrap@4.0.0-alpha.6
```
To add Bootstrap's assets to your project, open `angular-cli.json` in your app root and add the following:

```javascript
apps: [
  {
   //..
   "styles": [
        "../node_modules/bootstrap/dist/css/bootstrap.css"
    ],
    //...
    "environments": {
    //...
    "scripts": [
      "../node_modules/jquery/dist/jquery.js",
      "../node_modules/tether/dist/js/tether.js",
      "../node_modules/bootstrap/dist/js/bootstrap.js"
    ]
  }
```

This will let your `angular-cli` setup to locate the javascript and css files of your Boostrap installation and add them in the build of the project.

Next, add `ng-boostrap` to  your dependencies:
```
$ yarn add @ng-boostrap/ng-boostrap
```

And include the `NgbModule` in your app's root module (located in `app.module.ts`):
```
import {NgbModule} from "@ng-bootstrap/ng-bootstrap";

@NgModule({
    //..
    imports: [
        NgbModule
    ],
    //..
})
```

### Setting up the application store and meta reducer
 Next, we are going to make a bare minimum implementation of a Redux architecture that will serve as a foundaton of all the use cases that will be later implemented in this guide.

Start off by adding the core dependencies for the Redux application store:
```
$ yarn add @ngrx/core
$ yarn add @ngrx/store
```

For asynchronous events such as pagination and loading bars, in the layout of the application, there needs to be a middleware:
```
$ yarn add @ngrx/effects
```
To make selection of the state fast an efficient, add `reselect`. We are going to use `reselect`'s  <code>createSelector</code> function to create efficient selectors that are [memoized](https://en.wikipedia.org/wiki/Memoization) and only recompute when arguments change.
```
$ yarn add reselect
```
To make development more convenient and easier to debug, add the a store logger which will log to the console every action and the new state of the state.
```
$ yarn add ngrx-store-logger
```
To structure the application's files properly, all the redux-related files will stay in `src/app/common` directory.

```
$ mkdir src/app/common
```

#### Creating the layout state

Create `common/layout` directory which is going to contain all actions, effects, and the reducer of the layout sate. 

```
$ mkdir src/app/common/layout
```

In the directory create three files for the layout state:
```
$ touch layout.actions.ts 
```
** layout.actions.ts **

The layout actions will be dispatched every time an user action is made (closing and opening sidebar, opening a modal) or certain window events happen (window resizing).

```
import {Action} from '@ngrx/store';

/*
 Layout actions are defined here
 */


export const LayoutActionTypes = {


};

/*
 The action classes will be added here once they are defined
*/

export type LayoutActions = null;

```
**layout.reducer.ts**
```
$ touch layout.reducer.ts 
```
The reducer of the layout will handle all changes of the application layout and create a new state every time the layout changes.
```
import * as layoutActions from './layout.actions';

export interface State {
 /*
   The description of the different parts of the layout go here
  */
}

const initialState: State = {
  /*
    The initial values of the layout state will be initialized here
   */
};


/*
  The reducer of the layout state. Each time an action for the layout is dispatched,
  it will create a new state for the layout.
 */
export function reducer(state = initialState, action: layoutActions.LayoutActions): State {
  switch (action) {
    default:
      return state;
  }
}

```
#### Creating the meta reducer
With the layout state ready, the last step is to add the meta reducer, which will eventually be bootstrapped with the `StoreModule` provided by `@ngrx/store`.  If you are not very familiar with Redux and the role of the meta reducer, [read here](https://github.com/ngrx/example-app/blob/master/src/app/reducers/index.ts):

```
$ touch src/app/common/index.ts
```
```
/*
  Import createSelector from reselect to make selection of different parts of the state fast efficient
 */
import { createSelector } from 'reselect';
/*
  Import the store logger to log all the actions to the console
 */
import {storeLogger} from "ngrx-store-logger";

/*
 Import the layout state
 */

import * as fromLayout from "./layout/layout.reducer"
import {compose} from "@ngrx/core";
import {combineReducers} from "@ngrx/store";

export interface AppState {
  layout: fromLayout.State
}

export const reducers = {
  layout: fromLayout.reducer
};



const developmentReducer:Function = compose(storeLogger(), combineReducers)(reducers);


export function metaReducer(state: any, action: any) {
  return developmentReducer(state, action);
}

/**
 * Layout selectors
 */

export const getLayoutState = (state: AppState) => state.layout;

```
Finally, add the `metaReducer` to the `StoreModule` in the `imports` array of the root module:

```

import {StoreModule} from "@ngrx/store";
import {metaReducer} from "./common/index";
//...

@NgModule({
  //...
  imports: [
    //Provide the application reducer to the store.
    StoreModule.provideStore(metaReducer),
  ],
  //...
})
export class AppModule { }

```
# Modals

# Sidebars

# Pagination

# Accordion 

# Confirmation dialogs

# Window size

# Events

# Colors
