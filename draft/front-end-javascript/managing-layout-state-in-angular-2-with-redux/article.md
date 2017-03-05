# The benefits of controlling your application layout with Redux
 of the application is going to be a representation of the states of all elements in the layout - accordions, sidebars, pagination and everything that is pertinent to classify as part of the layout of the application. Reduxifying the layout leads to numerous benefits and adds great flexibility to controlling the layout of the applicaiton and make previously difficult use cases a breeze to implement.
 
 * Persist the state of your layout such as keeping the sidebar opened or closed when changing routes.
 * Control the layout in any point of the application, without worrying how components are related.
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
        NgbModule.forRoot()
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
$ cd src/app/common/layout
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
import * as layout from './layout.actions';

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
export function reducer(state = initialState, action: layout.LayoutActions): State {
  switch (action.type) {
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
#### Smart containers and directives, dumb components
If you are fimilar with Redux, you would know that there are two types of components - [presentional(dumb components) and container components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.e8sft6x0y). When building the layout state, it's best to keep the logic inside directives in order to keep the logic [DRY](http://deviq.com/don-t-repeat-yourself/). You don't have to write the same logic for a sidebar in every container component in your application.

#Note Another possibility is to keep the logic in the container and only in exceptional cases there's a need to put the logic inside a component that represents a layout element.

In this guide, the container component will be the `AppComponent`. In order to make the state of the application accessible in it and be able to dispatch actions, you have to import `layout.actions` and the root state:

```
import { Component } from '@angular/core';
import {Store} from "@ngrx/store";
import {Observable} from "rxjs";
/**
 * Import the root state in order to select parts of it.
 */
import * as fromRoot from './common/index';
/*
 * Import the layout actions to make dispatching from the component possible.
 */
import * as layout from './common/layout/layout.actions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  constructor(private store: Store<fromRoot.AppState>) {}

}

```

All dumb components will be kept in a separate directory named `components`:
```
$ mkdir src/app/components
```
# Modals

The easiest way to mplement a modal in the state is to keep its name as an identifier. Since only one modal can be opened at a time in the layout (unless you're trying to do some kind of black magic), every modal can be referenced by a `modalName`.

Let's start with the actions. An user can open and close a modal, so let's add actions for that:

### Adding to the state
**layout.actions.ts**
```
export const LayoutActionTypes =  {
  OPEN_MODAL: '[Layout] Open modal',
  CLOSE_MODAL: '[Layout] Close modal'
};

/*
  Modal actions
 */
export class OpenModalAction implements Action {
  type = LayoutActionTypes.OPEN_MODAL;
  constructor(public payload:string) {
  }
}

export class CloseModalAction implements Action {
  type = LayoutActionTypes.CLOSE_MODAL;
  constructor() {
  }
}


export type LayoutActions = CloseModalAction | OpenModalAction
```

Let's go forward and implement how modal actions will be handled in the layout reducer:
**layout.reducer.ts**
```
import * as layout from './layout.actions';

export interface State {
  openedModalName:string;
}

const initialState: State = {
  openedModalName: null
};


export function reducer(state = initialState, action: layout.LayoutActions ): State {
  switch (action.type) {
    /*
      Modal cases
     */
    case layout.LayoutActionTypes.OPEN_MODAL: {

      const name = action.payload;
      return Object.assign({}, state, {
        openedModalName:name
      });
    }

    case layout.LayoutActionTypes.CLOSE_MODAL: {
      return Object.assign({}, state, {
        openedModalName:null
      });
    }
    default:
      return state;
  }
}

export const getOpenedModalName = (state:State) =>  state.openedModalName;
```
The currently opened modal's name will be stored in the `openedModalName` which will be set and unset according to the dispatched action.
A selector `getOpenedModalName` is needed to easily access the `openedModalName` property within the state.

In `index.ts`, add a selector to access the `openedModalName` property from the application state:
**index.ts**
```
export const getLayoutState = (state: AppState) => state.layout;

//...
export const getLayoutOpenedModalName = createSelector(getLayoutState , fromLayout.getOpenedModalName);
```
### Usage 
 To see how it works, let's create a sample modal:
 
```
$ touch src/app/components/template-modal.component.ts
$ touch src/app/components/template-modal.template.ts
```

**template-modal.component.ts**
```

import {Component, ChangeDetectionStrategy, Output, ViewChild, EventEmitter, Input, ElementRef} from '@angular/core';
import {NgbModal, NgbModalRef} from "@ng-bootstrap/ng-bootstrap";


@Component({
  selector: 'template-modal',
  templateUrl: 'template-modal.template.html'
})
export class TemplateModalComponent {

  private modalName:string =  'templateFormModal';
  private modalRef:NgbModalRef;

  @ViewChild('content') _templateModal:ElementRef;

  @Input() set modalState(_modalState:any) {
    if(_modalState == this.modalName) {
      this.openModal()
    } else if(this.modalRef) {
      this.closeModal();
    }
  }

  @Output() onCloseModal = new EventEmitter<any>();

  constructor(private modalService: NgbModal) {}

  openModal() {
    this.modalRef = this.modalService.open(this._templateModal, {backdrop: 'static' , keyboard: false, size: 'sm'})
  }

  closeModal()  {
    this.modalRef.close();
  }


}

```
The name of the currently selected modal is coming from the container and is obtained through the `modalState` input of the component. If the name matches with the `modalName` (templateFormModal), then the modal is opened through the `modalService`.
Conversely, `onCloseModal` is used to emit to the container that te user has clicked to close the modal.

**template-modal.template.html**
```
<template #content="" let-c="close" let-d="dismiss">
  <div class="modal-header">
    <div class="row">
      <div class="col-sm-10">
        Template Modal
      </div>
      <div class="col-sm-2">
        <button class="close" type="button" aria-label="Close" (click)="d('Cross click'); onCloseModal.emit()"><span aria-hidden="true">×</span></button>
      </div>
    </div>
  </div>
  <div class="modal-body">
    Modal with Redux using a template. You can put anything here
  </div>
  <div class="modal-footer">
    <div class="btn-group">
      <button class="btn btn-warning" type="button" (click)="d('Cross click'); onCloseModal.emit()">Close</button>
    </div>
  </div>
</template>
```
Every time the user attempts to close te modal, `onCloseModal` directly emits from the template to the container. In case you have issues understanding how the modal works, you cna check the [standard implementation of a ng-bootstrap modal without Redux](https://ng-bootstrap.github.io/#/components/modal).

In the container there need to be handlers for gettng the `openedModalName` from the state and dispatching an action for closing a modal:

**app.component.ts**
```
export class AppComponent {
  public openedModalName$: Observable<any>;

  constructor(private store: Store<fromRoot.AppState>) {
    // Use the selector to directly get the opened modal name from the state
    this.openedModalName$ = store.select(fromRoot.getLayoutOpenedModalName);
  }

  //Dispatch an action to open a modal
  handleOpenModal(modalName:string) {
    this.store.dispatch(new layout.OpenModalAction(modalName));
  }

  handleCloseModal() {
    this.store.dispatch(new layout.CloseModalAction());
  }

}
```

As you can see, you can reuse the `handleOpenModal` and `handleCloseModal`. No matter how many modals your container has, the only thing that needs to be specified is the `modalName` of the modal you would like to see opened. 

**app.template.html**
```
<!-- Use the async pipe to get the latest broadcasted value of on observable as an input in the component  -->
<template-modal [modalState]="this.openedModalName$ | async" (onCloseModal)="handleCloseModal()"></template-modal>
<button class="btn btn-outline-primary" (click)="handleOpenModal('templateFormModal')">Open modal with template</button>

<!-- Don't forget to add this: -->
<template ngbModalContainer></template>

```
In this case `handleOpenModal` is dispatched through clicking a button, but it can also be dispatched as an output from another component, a directive,a service or an effect. The possibilities are endless.
# Sidebars

# Pagination

# Accordion 

# Confirmation dialogs

# Window size

# Events

# Colors
