Despite the numerous advances in building web interfaces in the last few years, the control of the DOM and the application User Interface (UI) is still highly dependent on [jQuery](https://jquery.com/), a 10-year-old library. Although this is not necessarily a bad thing, jQuery was initially built for different purposes than it's used for nowadays. As a result, modern uses have started to cause issues. Front-end applications have become increasingly complex, with all kinds of unrelated components, encapsulated views and other elements trying to interact together. 
 
In this guide, we'll explore using Redux to mitigate the current challenges in controlling the UI of an Angular 2 application. You will learn how to represent some of the logic for controlling the UI layout in reducer functions and see an example use case.

###  Taming your application UI layout

Since Redux came out, state management for front-end apbplications went through a revolution. My team and I have field-tested Redux with Angular 2 and the resulting productivity boost has been tremendous. 
> Redux has not only allowed us to ship faster, but it increased the overall maintainability of our codebase by encapsulating most of the crucial logic in a single place and providing an easy to test architecture. 

We liked Redux so much that we wanted to control _everything_ with it. One of our most recent projects was building a very UI-intensive application and we decided to experiment by giving the reducers in the applicaiton a little bit more resposibility than just controlling data.

We found out that "reduxifying" the UI leads to numerous benefits and makes controlling the flexibility . It made previously difficult use cases a breeze to implement.

##### *Three key benefits of using Redux for your UI layout* 

 * ** Maintain the state of your UI such as keeping the sidebar opened or closed when changing routes.**
 * **Control the UI from any point of the application without worrying about how components are related or how to inject a specific service.**
 * **Chain UI layout-specific actions with other events such as saving data server-side or changing a route**

# Setup 
 > The examples  below will be done with ng-bootstrap since it's one of the most popular libraries with Angular 2 components for Bootstrap 4. You can also implement these examples with other component libraries, such as [Material Design](https://github.com/angular/material2), by following the same design principles and making small adjustments to the code so that it works with the API of the corresponding library.

### Dependencies
 Below are the packages you need to install in order to start working:

**Redux**
 - [@ngrx/store + @ngrx/core](https://github.com/ngrx/store)
 - [@ngrx/effects](https://github.com/ngrx/effects)
 - [reselect](https://github.com/reactjs/reselect)
 - [ngrx-store-logger](https://github.com/btroncone/ngrx-store-logger)

** Bootstrap **
 - [Bootstrap 4](https://github.com/twbs/bootstrap)
 - [ng-bootstrap](https://github.com/ng-bootstrap/ng-bootstrap)

### Installing

To ensure a smooth setup, we will use [Angular CLI](https://github.com/angular/angular-cli) to initialize the application architecture. Make sure you have it installed globally before you proceed. 

In your terminal, write the following commands to initialize your Angular 2 app:

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

This will let your `angular-cli` locate the JavaScript and CSS files of your Bootstrap installation and add them in the build of the project.

Next, add `ng-bootstrap` to  your dependencies:
```
$ yarn add @ng-bootstrap/ng-bootstrap
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
To make development more convenient and easier to debug, add a *store logger*, which will log to the console every action and the new state of the state.
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

The layout actions will be dispatched every time when an user action is made (closing and opening sidebar, opening a modal and so on) or when certain events happen (window resizing).

```
import {Action} from '@ngrx/store';

/*
 Layout actions are defined here
 */

export const LayoutActionTypes = {};

/*
 The action classes will be added here once they are defined
*/
export type LayoutActions = null;

```
**layout.reducer.ts**
```
$ touch layout.reducer.ts
```
The reducer of the layout will handle all changes of the application layout and create a new state every time the UI has to change.
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
With the UI state ready, the last step is to add the meta reducer, which will eventually be bootstrapped with the `StoreModule` provided by `@ngrx/store`.  If you are not very familiar with Redux and the role of the meta reducer, [read here](https://github.com/ngrx/example-app/blob/master/src/app/reducers/index.ts):

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
    reducer: {
        layout: fromLayout.State;
    };
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

export const getLayoutState = (state: AppState) => state.reducer.layout;

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
    StoreModule.forRoot({ reducer: metaReducer }),
  ],
  //...
})
export class AppModule { }

```
#### "Smart" containers and "dumb" components
If you are fimilar with Redux, you would know that there are two types of components - [presentational components and container components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.e8sft6x0y). 

> When building the UI state, it's best to keep the logic inside [directives](https://angular.io/guide/attribute-directives) in order to keep the logic [DRY](http://deviq.com/don-t-repeat-yourself/). You don't have to write the same logic for a sidebar in every container component in your application.

Another possibility is to keep the logic in the container and only in exceptional cases there's a need to put the logic inside a component that represents a UI element.

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


# Modals

The easiest way to implement a modal in the state is to keep its name as an identifier. Since only one modal can be opened at a time in the UI view (unless you're trying to do some kind of black magic), every modal can be referenced by a `modalName`.

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
  constructor(public payload:string) {
  }
}


export type LayoutActions = CloseModalAction | OpenModalAction
```

Let's go ahead and implement how modal actions will be handled in the layout reducer:
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
The currently modal's name will be stored in the `openedModalName` which will be set and unset according to the dispatched action. A selector `getOpenedModalName` is needed to easily access the `openedModalName` property within the state.

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
$ ng g component template-modal
```
**template-modal.component.ts**
```

import {Component, ChangeDetectionStrategy, Output, ViewChild, EventEmitter, Input, ElementRef} from '@angular/core';
import {NgbModal, NgbModalRef} from "@ng-bootstrap/ng-bootstrap";


@Component({
  selector: 'template-modal',
  templateUrl: 'template-modal.component.html'
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
The name of the currently-selected modal is coming from the container and is obtained through the `modalState` input of the component. If the name matches with the `modalName` (templateFormModal), then the modal is opened through the `modalService`.

Conversely, `onCloseModal` is used to emit to the container that te user has clicked to close the modal.

**template-modal.component.html**
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
Every time the user attempts to close the modal, `onCloseModal` directly emits from the template to the container. In case you have issues understanding how the modal works, you can check the [standard implementation of a ng-bootstrap modal without Redux](https://ng-bootstrap.github.io/#/components/modal).

In the container there need to be handlers for getting the `openedModalName` from the state and for dispatching an action for closing a modal:

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

> As you can see, you can reuse the `handleOpenModal` and `handleCloseModal`. No matter how many modals your container has, the only thing that needs to be specified is the `modalName` of the modal you would like to see opened.

**app.component.html**
```
<!-- Use the async pipe to get the latest broadcasted value of on observable as an input in the component  -->
<template-modal [modalState]="this.openedModalName$ | async" (onCloseModal)="handleCloseModal()"></template-modal>
<button class="btn btn-outline-primary" (click)="handleOpenModal('templateFormModal')">Open modal with template</button>

<!-- Don't forget to add this: -->
<template ngbModalContainer></template>

```
In this case `handleOpenModal` is dispatched by clicking a button, but it can also be dispatched as an output from another component, a directive,a service or an effect. The possibilities are endless.


![modal example](https://raw.githubusercontent.com/pluralsight/guides/master/images/f0723337-3b32-4649-9d97-93bab2145b09.gif)


# Sidebar(s)
 The most generic representation of a sidebar in an application comes down to whether the sidebar is opened or not. The state will have a property that denotes whether a sidebar is `opened` or `closed` using a boolean value. In case there are two sidebars (or more, depends on what kind of sorcery your're doing), there will be a property in the state for each.

 For the user to start interacting with the sidebar, there need to be actions for opening and closing:

 **layout.actions.ts**
 ```
 export const LayoutActionTypes =  {
  //Left sidenav actions
  OPEN_LEFT_SIDENAV: '[Layout] Open LeftSidenav',
  CLOSE_LEFT_SIDENAV: '[Layout] Close LeftSidenav',
  //Right sidenav actions
  OPEN_RIGHT_SIDENAV: '[Layout] Open RightSidenav',
  CLOSE_RIGHT_SIDENAV: '[Layout] Close RightSidenav',
};


export class OpenLeftSidenavAction implements Action {
  type = LayoutActionTypes.OPEN_LEFT_SIDENAV;

  constructor() {
  }
}
export class CloseLeftSidenavAction implements Action {
  type = LayoutActionTypes.CLOSE_LEFT_SIDENAV;

  constructor() {
  }
}
export class OpenRightSidenavAction implements Action {
  type = LayoutActionTypes.OPEN_RIGHT_SIDENAV;

  constructor() {
  }
}

export class CloseRightSidenavAction implements Action {
  type = LayoutActionTypes.CLOSE_RIGHT_SIDENAV;

  constructor() {
  }
}


export type LayoutActions =  CloseLeftSidenavAction | OpenLeftSidenavAction |   CloseRightSidenavAction |OpenRightSidenavAction
 ```
 As mentioned, the states of the sidebar will be represented with booleans. In this case, the left sidenav will be open by default, but there can be logic that checks if the window size is small enough to dispatch a `CloseLeftSidenavAction` to close it:

 **layout.reducer.ts**
 ```
 import * as layout from './layout.actions';

export interface State {;
  leftSidebarOpened:boolean;
  rightSidebarOpened:boolean;
}

const initialState: State = {
  leftSidebarOpened:true,
  rightSidebarOpened:false
};


export function reducer(state = initialState, action: layout.LayoutActions ): State {
  switch (action.type) {
    case layout.LayoutActionTypes.CLOSE_LEFT_SIDENAV: {
      return Object.assign({}, state, {
        leftSidebarOpened: false
      });
    }
    case layout.LayoutActionTypes.OPEN_LEFT_SIDENAV: {
      return Object.assign({}, state, {
        leftSidebarOpened: true
      });
    }
    case layout.LayoutActionTypes.CLOSE_RIGHT_SIDENAV: {
      return Object.assign({}, state, {
        rightSidebarOpened: false
      });
    }
    case layout.LayoutActionTypes.OPEN_RIGHT_SIDENAV: {
      return Object.assign({}, state, {
        rightSidebarOpened: true
      });
    }

    default:
      return state;
  }
}

export const getLeftSidenavState = (state:State) => state.leftSidebarOpened;
export const getRightSidenavState = (state:State) => state.rightSidebarOpened;
```
In the root of the state, add selectors to access the states of the sidebars:
**index.ts**
```
//...

export const getLeftSidenavState = (state:State) => state.leftSidebarOpened;
export const getRightSidenavState = (state:State) => state.rightSidebarOpened;

```

### Usage


 Instead of putting the logic in each of the sidebar components, it can be combined within a structural directive that closes and opens the corresponding sidebar depening on the state of the application.

 ```
 $ ng g directive sidebar-watch
 ```

 **sidebar-watch.directive.ts**

 ```
import {Directive, ElementRef, Renderer, OnInit, AfterViewInit, AfterViewChecked} from '@angular/core';
import {Store} from "@ngrx/store";
import * as fromRoot from "../common/index";
let $ = require('jquery');

@Directive({selector: '[sidebarWatch]'})

export class SidebarWatchDirective implements OnInit{
  constructor(private el: ElementRef, private _store: Store<fromRoot.AppState>) {}

  /*
   Doing the checks on ngOnInit makes sure the DOM is fully loaded and the
   elements are available to be selected
   */
  ngOnInit() {
    /*
     Watch for the left sidebar state
     */
    this._store.select(fromRoot.getLayoutLeftSidenavState).subscribe((state) => {
      if (this.el.nativeElement.className == 'left-sidebar') {
        if (state) {
          $("#main-content").css("margin-left", "300px");
          $(this.el.nativeElement).css('width', '300px');
        } else {
          $("#main-content").css("margin-left", "0");
          $(this.el.nativeElement).css('width', '0');
        }
      }
    });

    /*
     Watch for the right sidebar state
     */
    this._store.select(fromRoot.getLayoutRightSidenavState).subscribe((state) => {
      /*
       You can use classes (addClass/removeClass) instead of using jQuery css(), or you
       can go completely vanilla by using selectors such as windiw.getElementById(). .
       */
      if (this.el.nativeElement.className == 'right-sidebar') {
        console.log('test')
        if (state) {
          $('#fade').addClass('fade-in');
          $("#rightBar-body").css("opacity", "1");
          $("body").css("overflow","hidden");
          $(this.el.nativeElement).css('width', '60%');
        } else {
          $('#fade').removeClass('fade-in');
          $("#rightBar-body").css("opacity", "0");
          $("body").css("overflow","auto");
          $(this.el.nativeElement).css('width', '0');
        }
      }
    });
  }

}

 ```
 The directive checks `ElementRef`'s `nativeElement`, which make the DOM properties of the template accessible in the component. Once it knows which sidebar it is being applied to, the directive checks whether the corresponding state (`LeftSidenavbarState` or `RightSidenavbarState`, respectively) is `true` or `false`. Then the directive uses jQuery to manipulate the corresponding elements in the layout. The use of jQuery to select and directly change the DOM element's style properties is optional and you can use addition and removal of classes or by using plain JavaScript.

 Following the same logic, there can be a directive for toggling the sidebars:

 ```
  $ ng g directive sidebar-toggle
 ```

 **sidebar-toggle.directive.ts**

 ```
 /**
 * Created by Centroida-2 on 1/22/2017.
 */
import {Directive, Input, ElementRef, Renderer, HostListener} from '@angular/core';
import {Store} from "@ngrx/store";
import * as fromRoot from "../common/index";
import * as layout from '../common/layout/layout.actions'
@Directive({
  selector: '[sidebarToggle]'
})

export class SidebarToggleDirective {
  public leftSidebarState: boolean;
  public rightSidebarState: boolean;
  @Input() sidebarToggle: string;

  @HostListener('click', ['$event'])
  onClick(e) {
    /*
     Left sidenav toggle
     */
    if (this.sidebarToggle == "left" && this.leftSidebarState) {
      this._store.dispatch(new layout.CloseLeftSidenavAction());
    } else if(this.sidebarToggle == "left" && !this.leftSidebarState) {
      this._store.dispatch(new layout.OpenLeftSidenavAction())
    }

    /*
     Right sidenav toggle
     */
    if (this.sidebarToggle == "right" && this.rightSidebarState) {
      this._store.dispatch(new layout.CloseRightSidenavAction());
    } else if(this.sidebarToggle == "right" && !this.rightSidebarState) {
      this._store.dispatch(new layout.OpenRightSidenavAction());
    }
  }

  constructor(private el: ElementRef,private renderer: Renderer, private _store: Store<fromRoot.AppState>) {
    this._store.select(fromRoot.getLayoutLeftSidenavState).subscribe((state) => {
      this.leftSidebarState = state;
    });

    this._store.select(fromRoot.getLayoutRightSidenavState).subscribe((state) => {
      this.rightSidebarState = state;
    });
  }

}

 ```

 The directive has an `@Input sidebarToggle` which can be either `left` or `right` , depending on which sidebar the directive has to control.  Every time the user clicks on the element to which the directive is attached, the `@HostListener('click')` catches the event and checks the state of the sidebar of the store and dispatches the corresponding action.


To demonstrate how everything comes together, let's make two sidebars:
```
$ ng g component left-sidebar
```

**left-sidebar.component.ts**

```
import { Component } from '@angular/core';

@Component({
  selector: 'left-sidebar',
  templateUrl: 'left-sidebar.component.html',
  styleUrls: ['./sidebar.styles.css']
})
export class LeftSidebarComponent  {
  constructor() {}
}
```

**left-sidebar.component.html**
```
<section sidebarWatch class="left-sidebar">
</section>
```

```
$ ng g component right-sidebar
```
**right-sidebar.component.ts**
```
import { Component } from '@angular/core';

@Component({
  selector: 'right-sidebar',
  templateUrl: 'right-sidebar.component.html',
  styleUrls: ['./sidebar.styles.css']
})
export class RightSidebarComponent  {
  constructor() {}
}
```

**right-sidebar.component.html**
```
<section sidebarWatch class="right-sidebar">
  <button class="btn btn-primary" sidebarToggle="right">Close Right Sidebar</button>
</section>
```
Using the `sidebarWatch` directive is quite straightforward. Just put in the topmost elements of the sidebars.

The `sidebarToggle` needs to be put in a button (although you can put it anywhere you like) and it needs to have the `left` or `right` value assigned to it.

To make the elements look and feel like sidebars, there needs to be some additional CSS:

```
$ touch src/app/components/sidebar.styles.css
```

**sidebar.styles.css**

```
.left-sidebar,.right-sidebar {
  transition: width 0.3s;
  height: 100%;
  position: fixed;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.15);
}

.left-sidebar{
  background: #909090;
}
.right-sidebar{
  overflow-y: auto !important;
  overflow-x: hidden !important;
  right: 0;
  z-index: 999 !important;
  background:#212121;

}
```



In the root component of the application, put the sidebars on top all the content in a `div` with a class `main-content`:
**app.component.html**

```
<div id="fade" class="fade-in"></div>
<left-sidebar></left-sidebar>
<right-sidebar></right-sidebar>
<div id="main-content">
  <button class="btn btn-primary" sidebarToggle="left">Toggle Left Sidebar</button>
  <button class="btn btn-primary" sidebarToggle="right">Toggle Right Sidebar</button>

  <!-- ... -->
</div>
<!-- ... -->
```
The div with id `fade` will be used for the fade effect when the right sidebar is opened. Add the styles to the component styles:

**app.component.css**

```
.fade-in {
  position: absolute;
  min-height: 100%!important;
  top: 0; right: 0; bottom: 0; left: 0;
  background: rgba(0,0,0,0.5);
  width: 100%;
  transition: top 0.3s, right 0.3s, bottom 0.3s, left 0.3s;
}

```

> With this setup, the sidebars are truly container-agnostic. Any element can be made a sidebar through a directive and be toggled from any point of the UI layout. There is also flexibility if there's a requirement  to add additional components such as bottom bars or top bars.


![sidebars example](https://raw.githubusercontent.com/pluralsight/guides/master/images/d183dadf-0013-4a7a-a719-5558681b35c1.gif)



# Dismissable Alerts

 Including alerts in the application state gives control when, where and how alerts can appear. Since alerts are dependent on either server-side or user actions, their place in the application state is well-deserved.
 
 Unlike other examples in this guide, "reduxifying" alerts is somehwat easy - they can be simply  represented as a local collection of items that can be added and removed from the state. 
 
 By default, an alert would have two attributes: `message` and `type`. Here is how the model of a alert would look like:
 
 ```
 export class Alert {
   message:string;
   type:string
 }

 ```
 
First, let's  add actions for removing and adding alerts:
 
 **layout.actions.ts**
 
 ```
 export const LayoutActionTypes =  {
  ADD_ALERT: '[Layout] add alert',
  REMOVE_ALERT: '[Layout] remove alert'
  //...
};

//...
export class AddAlertAction implements Action {
  type = LayoutActionTypes.ADD_ALERT;
  constructor(public payload:Object) {
  }
}

export class RemoveAlertAction implements Action {
  type = LayoutActionTypes.REMOVE_ALERT;
  constructor(public payload:Object) {
  }
}
//...
 
export type LayoutActions =  AddAlertAction | RemoveAlertAction

 
```
Second, let's add the `alerts` slice in the `layout` state:

**layout.reducer.ts**
```
import * as layout from './layout.actions';

export interface State {
  //...
  alerts:Array<Object>;
}

const initialState: State = {
  //...
  alerts:[],
};


export function reducer(state = initialState, action: layout.LayoutActions ): State {

  switch (action.type) {
    case layout.LayoutActionTypes.ADD_ALERT: {
      return Object.assign({}, state, {
        alerts:[...state.alerts, action.payload]
      });
    }
    case layout.LayoutActionTypes.REMOVE_ALERT: {
      return Object.assign({}, state, {
        /*
         Alerts are filtered by message content, but for real-world usage, an 'id' field would be more suitable.
        */
        alerts: state.alerts.filter(alert =>
          alert['message'] !== action.payload['message']
        )
      });
    }
    //...
    default:
      return state;
  }
}

//...
/*
 If you add more attributes to the alerts such as 'position' or 'modelType',
 there can be more selectors added that can filter the collection and allow
 only certain to be displayed in designated places in the application.
*/
export const getAlerts = (state:State) => state.alerts;
```
Finally, let's add a selector for `alerts` in the root:

**index.ts**
```
//...
export const getLayoutAlertsState = createSelector(getLayoutState, fromLayout.getAlerts);
```

That's it. Now alerts are part of the application state. But how are they going to be used? Let's find out:

### Usage

With the tools given, making dismissible alerts requires a very small amount of code since [ng-bootstrap](https://ng-bootstrap.github.io/#/components/alert) already offers an implementation. Thus, the only thing that is required is a reusable component to be made that can be used in various places in the application:

```
$touch src/app/alerts-list.component.ts
```
**alerts.component.ts**
```

import {Component, Input, EventEmitter, Output} from '@angular/core';

@Component({
  selector: 'alerts-list',
  templateUrl: 'alerts-list.component.html',

})
export class AlertsListComponent  {
  @Input() alerts:any;
  @Output() closeAlert = new EventEmitter();
  
  constructor() {
  }
}
```

The component accepts an array of alerts and outputs the alert which the user decides to close.

```
$touch src/app/alerts-list.component.html
```
**alerts.component.html**
```
<p *ngFor="let alert of alerts">
  <ngb-alert [type]="alert.type" (close)="closeAlert.emit(alert)">{{ alert.message }}</ngb-alert>
</p>
```

Don't forget to add the component to the root module:

**app.module.ts**
```
import {AlertsListComponent} from "./components/alerts-list.component";
///...

@NgModule({
  declarations: [

    AlertsListComponent,
  ],
  //...
})
export class AppModule { }

```

Next, the logic in the container component has to be implemented to select alerts and dispatch events.

**app.component.ts**
```
import {Component, OnInit} from '@angular/core';
import {Store} from "@ngrx/store";
import {Observable} from "rxjs";
import * as fromRoot from './common/index';
import * as layout from './common/layout/layout.actions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',

})
export class AppComponent implements  OnInit{
  public alerts$:Observable<any>;

  constructor(private store: Store<fromRoot.AppState>) {
    this.alerts$ = store.select(fromRoot.getLayoutAlertsState);
  }

  addAlert(alert) {
    this.store.dispatch(new layout.AddAlertAction(alert))
  }

  onCloseAlert(alert:Object) {
   this.store.dispatch(new layout.RemoveAlertAction(alert))
  }
}

```

To demonstrate how alerts look, the template will have two buttons for opeaning different types of alerts:

**app.component.html**

```
<div id="fade" class="fade-in"></div>
<left-sidebar></left-sidebar>
<right-sidebar></right-sidebar>

<div id="main-content">
    <!-- List of alerts goes here -->
    <alerts-list [alerts]="alerts$ | async (closeAlert)="onCloseAlert($event)"></alerts-list>
    
    <!-- Buttons for creating alerts -->
    <button class="btn btn-danger" (click)="addAlert({type: 'danger', message: 'This is a danger alert'})">Add a danger alert</button>
    <button class="btn btn-success" (click)="addAlert({type: 'success', message: 'This is a success alert'})">Add a success alert</button>
</div>



```


![alert example](https://raw.githubusercontent.com/pluralsight/guides/master/images/1fde8c90-c987-4760-9198-7c33727710a7.gif)

In a real-world scenario, alerts can be created when a server returns certain results. For example, in the snippet below, an `AddAlertAction` is called once the application resolves a server-side request:

```
  @Effect() deleteStudent = this._actions.ofType(student.ActionTypes.DELETE_STUDENT)
    .switchMap((action) => this._service.delete(action.payload)
    )
    .mergeMap(
      () => {
        return Observable.from([
          new DeleteStudentSuccessAction(),
          /* Chain actions - once the server successfully
           deletes some model, create an alert from it.
          */
          new layout.AddAlertAction({type:'success', message: 'Student successfully deleted!')
        ])
      .catch(() => {
             new layout.AddAlertAction({type:'danger', message: 'An error ocurred.'})
             return Observable.of( new DeleteStudentFailureAction()
            })
        );
      }
    );
```

# Window size
Having the window size available in the application store can make Redux useful for numerous use cases, especially for making responsive UI changes, device-specific actions or dynamic changes of the CSS (using [NgClass](https://angular.io/docs/ts/latest/api/common/index/NgClass-directive.html) or [NgStyle](https://angular.io/docs/ts/latest/api/common/index/NgStyle-directive.html) ).

To make the window size usable in the application state, it has to be updated every time the window is resized. Let's add an action for that:

**layout.actions.ts**
```
import {Action} from '@ngrx/store';

export const LayoutActionTypes =  {
  // Add indow resize action
  RESIZE_WINDOW: '[Layout] Resize window'
};

export class ResizeWndowAction implements Action {
  type = LayoutActionTypes.RESIZE_WINDOW;
  constructor(public payload:Object) {
  }
}

export type LayoutActions = ResizeWndowAction
```

To implement the window size in the UI state, there need to be two attributes added to the state - `windowWidth` and `windowHeight`:

**layout.reducer.ts**
```
import * as layout from './layout.actions';

export interface State {
  //...
  windowHeight:number;
  windowWidth:number;
}

const initialState: State = {
  //...
  windowHeight: window.screen.height,
  windowWidth: window.screen.width
};

export function reducer(state = initialState, action: layout.LayoutActions ): State {
  switch (action.type) {
    /*
     Window resize case
     */
    case layout.LayoutActionTypes.RESIZE_WINDOW: {
      const height:number = action.payload['height'];
      const width:number = action.payload['width'];
      return Object.assign({}, state, {
        windowHeight: height,
        windowWidth: width
      });
    }
    //...

    default:
      return state;
  }
}

export const getWindowWidth = (state:State) => state.windowWidth;
export const getWindowHeight = (state:State) => state.windowHeight;

```

The inital state is set by using `window.screen.height` and `window.screen.width`, which get the values of the window's size when the state is initialized. The `WindowResizeAction` payload contains an object with the height and the width of the resized window:
`{width:number , height:number}`.

There are numerous ways to listen for window resize changes, but perhaps one of the most convenient and conventional ones is to put a `host` attribute in the application's root component decorator (`Appcomponent`). Since the listener is attached to the root component, `ResizeWindowAction` will be dispatched regardless of which part in the application the user is.

**app.component.ts**

```
import {Component, OnInit} from '@angular/core';
import {Store} from "@ngrx/store";
import {Observable} from "rxjs";
import * as fromRoot from './common/index';
import * as layout from './common/layout/layout.actions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  /*
   Add this to your AppComponent to listen for window resize events
   */
  host: {
    '(window:resize)': 'onWindowResize($event)'
  }
})
export class AppComponent implements  OnInit{
  //...
  constructor(private store: Store<fromRoot.AppState>) {
   //...
  }

  ngOnInit() {
  }
  
  onWindowResize(event){
    this.store.dispatch(new layout.ResizeWndowAction({width:event.target.innerWidth,height:event.target.innerHeight }))
  }

}
```
The `host` listens for window resize events and calls the `onWindowResize` method with the event as a parameter. The method gets the new sizes using the `event.target` property and dispatches a `ResizeWindowAction` with the new values.

### Usage 

The most obiquitous case for using the window size is responsiveness. For example, suppose the left sidebar has to automatically close if the window width is lower than 768px (iPad width). Doing this with Redux is quite simple - just add an if statement to the corresponding case:

**layout.reducer.ts**
```
//...

export function reducer(state = initialState, action: layout.LayoutActions ): State {
  switch (action.type) {
    //...
    case layout.LayoutActionTypes.RESIZE_WINDOW: {
        const height:number = action.payload['height'];
        const width:number = action.payload['width'];

        // If the width is lower than 768px, assign false. Otherwise don't change the state
        const leftSidebarState = width < 768 ? false : state.leftSidebarOpened;

        return Object.assign({}, state, {
          windowHeight: height,
          windowWidth: width,
          leftSidebarOpened: leftSidebarState
        });
      }
    }
 //...
}
```

The equivalent jQuery operation would be quite frustrating. However, with Redux, simply adding a simple ternary operator in the reducer does the trick. 
> With Redux, all the logic is isolated in a single place, and it is easy to debug and test all your implementations. 


![window resize example](https://raw.githubusercontent.com/pluralsight/guides/master/images/af6575a9-50ff-4d9a-aaae-b43861b2e265.com-video-to-gif)




# Server-side Pagination 

 Even though pagination is not strictly part of an application's UI layout, it is an integral part of the application's UI that can be implemented with Redux. The goal of implementing server-side pagination is to utilize the application store as much as possible and achieve flexibility with
 the least code possible. 
 
 ## GiantBomb API
 
 To illustrate how Redux pagination works, we will use the [GiantBomb API](http://www.giantbomb.com/api/) as the source of information. We will fetch the games stored in the GiantBomb database, and then we will paginate the results. The pagination will be controlled by the application state.

 First, create a separate directory for `games`:
 ```
 $ mkdir src/app/common/games
 ```

 ```
  $ touch src/app/common/games.actions.ts
 ```

 **games.actions.ts**

 ```
import {type} from "../util";
import {Action} from "@ngrx/store";
export const GameActionTypes =  {
  /*
   Because the games collection is asynchronous, there need to be actions to handle
   each of the stages of the request.
   */
  LOAD: '[Games] load games',
  LOAD_SUCCESS: '[Games] successfully loaded games',
  LOAD_FAILURE: '[Games] failed to load games',
};

export class LoadGamesAction implements Action {
  type = GameActionTypes.LOAD;
  constructor(public payload:any) {}
}

export class LoadGamesFailedAction implements Action {
  type = GameActionTypes.LOAD_FAILURE;

  constructor() {
  }
}
export class LoadGamesSuccessAction implements Action {
  type = GameActionTypes.LOAD_SUCCESS;
  constructor(public payload:any) {
  }
}

export type GameActions = LoadGamesAction | LoadGamesFailedAction | LoadGamesSuccessAction



 ```

Redux has a convention for loading asynchronous results. It does it by using three actions - `LOAD` , `LOAD_SUCCESS` and `LOAD_FAILURE`. The last two get dispatched when the [middleware](http://redux.js.org/docs/advanced/Middleware.html) resolves the server-side request.

To figure out how to construct the state of the paginated `games` entities, let's see what a pagination needs:

1. Number of current pages
2. Total amount of items
3. Collection of the items currently displayed
4. (optional) Number of items per page and number of visible pages


Having this in mind, here's how the state interface should look:

```
export interface State {
  loaded: boolean;
  loading: boolean;
  entities: Array<any>;
  count: number;
  page: number;
};
```

Let's see how the full implementation looks like:

```
$ touch src/app/common/games.reducer.ts
```

**games.reducer.ts**
```
import { createSelector } from 'reselect';
import * as games from './games.actions';

export interface State {
  loaded: boolean;
  loading: boolean;
  entities: Array<any>;
  count: number;
  page: number;
};

const initialState: State = {
  loaded: false,
  loading: false,
  entities: [],
  count: 0,
  page: 1
};

export function reducer(state = initialState, action: games.GameActions): State {
  switch (action.type) {
    case games.GameActionTypes.LOAD: {
      const page = action.payload;

      return Object.assign({}, state, {
        loading: true,
        /*
         If there is no page selected, use the page from the initial state
         */
        page: page == null ? state.page : page
      });
    }

    case games.GameActionTypes.LOAD_SUCCESS: {
      const games = action.payload['results'];
      const gamesCount = action.payload['number_of_total_results'];

      return Object.assign({}, state ,{
        loaded: true,
        loading: false,
        entities: games,
        count: gamesCount
      });
    }

    case games.GameActionTypes.LOAD_FAILURE: {
      return Object.assign({}, state ,{
        loaded: true,
        loading: false,
        entities:[],
        count: 0
      });
    }
    default:
      return state;
  }
}
/*
 Selectors for the state that will be later
 used in the games-list component
 */
export const getEntities = (state:State) =>  state.entities;
export const getPage = (state:State) => state.page;
export const getCount = (state:State) => state.count;
export const getLoadingState = (state:State) => state.loading;

```

Every time `LOAD` is called from the `GamesActions`, the page number is contained within the action's payload
and it is then assigned to the state. What's left is to find a way to query the server
using `page` from the games state. To do this, the state has to be added to the application store:



**index.ts**
```
import * as fromGames from "./games/games.reducer"
//...
export interface AppState {
  layout: fromLayout.State;
  games: fromGames.State
}

export const reducers = {
  layout: fromLayout.reducer,
  games: fromGames.reducer
};

//...
/*
 Games selectors
 */
export const getGamesState = (state: AppState) => state.games;
export const getGamesEntities = createSelector(getGamesState, fromGames.getEntities);
export const getGamesCount = createSelector(getGamesState, fromGames.getCount);
export const getGamesPage = createSelector(getGamesState, fromGames.getPage);
export const getGamesLoadingState = createSelector(getGamesState, fromGames.getLoadingState);


```

`getGamesPage` will be used to obtain the current page and send it as a parameter in the query to the service.
 
 ```
 $ touch src/app/common/games.service.ts
 ```
 
 **games.service.ts**
```
import {Injectable, Inject} from '@angular/core';
import {Response, Http, Headers, RequestOptions, Jsonp} from "@angular/http";
import {Store} from "@ngrx/store";
import * as fromRoot from "../index"


@Injectable()
export class GamesService {

  public page:number;

  constructor(private jsonp:Jsonp, private store: Store<fromRoot.AppState>) {
    /*
    Get the page from the games state
     */
    store.select(fromRoot.getGamesPage).subscribe((page) => {
      this.page = page;
    });
  }

  /*
  Get the list of games. GiantBomb requires a jsnop request with a token. You can use this token
  as a present from me, the author, and use it in moderation!
   */
  query() {
      let pagination = this.paginate(this.page);
      let url = `http://www.giantbomb.com/api/games/?api_key=b89a6126dc90f68a87a6fe1394e64d7312b242da&?&offset=${pagination.offset}&limit=${pagination.limit}&format=jsonp&json_callback=JSONP_CALLBACK`;
     return  this.jsonp.request(url, { method: 'Get' })
        .map((res) => {
          return res['_body']
        });
  }
  /**
   * This function converts a page to a pagination
   * query.
   *
   * @param page
   *
   * @returns {{offset: number, limit: number}}
   */

   paginate(page:number,){
      let beginItem: number;
      let endItem:number;
      // Items per page are hardcoded, but you can make them dynamic by adding another parameter
      let itemsPerPage:number = 10;
      if(page == 1 ){
        beginItem = 0;
      } else {
        beginItem = (page - 1) * itemsPerPage;
      }
      return {
        offset:beginItem,
        limit:itemsPerPage
      }
  }

}

```

The currently selected page is taken from the state and passed through `paginate`. `paginate` is a utility function that converts the current page to `offset` and `limit` parameters in accordance with the [GiantBomb API requirements for paginating results](http://www.giantbomb.com/api/documentation#toc-0-15).

Next, let's implement the middleware that will be used to call the service and dispatch
`SUCCESS` or `FAILURE` actions.

```
$ touch src/app/common/games.effects.ts
```

**games.effects.ts**

```
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';
import 'rxjs/add/operator/switchMap';
import { Observable } from 'rxjs/Observable';
import {Injectable} from "@angular/core";
import * as games from "./games.actions";
import {Actions, Effect} from "@ngrx/effects";
import {GamesService} from "./games.service";
import {LoadGamesSuccessAction} from "./games.actions";
import {LoadGamesFailedAction} from "./games.actions";

@Injectable()
export class GameEffects {
  constructor(
    private _actions: Actions,
    private _service: GamesService
  ) { }

  @Effect() loadGames$ = this._actions.ofType(games.GameActionTypes.LOAD)
    .switchMap(() => this._service.query()
      .map((games) => {
    return new LoadGamesSuccessAction(games)

  }))
    .catch(() => Observable.of( new LoadGamesFailedAction())
    );
}

```


Lastly, import the `EffectsModule` from `ngrx/effects` and run the effects and
add `GamesService` as a provider:

**app.module.ts**
```

import {EffectsModule} from "@ngrx/effects";
import {GameEffects} from "./common/games/games.effects";
import {GamesService} from "./common/games/games.service";
//...
@NgModule({
  //...
  imports: [
   //...
    EffectsModule.run(GameEffects)
  ],
  providers: [GamesService],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
This implementation of pagination provides a great deal of convenience and efficiency -  the application state is used to both to represent the state in the client and also give instructions to the server what results to fetch.
 
 ### Usage
 
 To demostrate the requirements for making a reusable and paginatable list component and to see the pagination in action, we will implement a `games-list` component.
 
 As mentioned earlier, four "slices" of a state need to be present for pagination to be possible:
 
 1. Collection of entities
 2. Total number of entities
 3. Current page
 4. Loading/Loaded status
 
Let's create the template of the `games-list` component first:
 
```
$ ng g component games-list
 ```
 
 **games-list.component.ts**
```
import {Component, OnInit, Input, EventEmitter, Output} from '@angular/core';

@Component({
  selector: 'games-list',
  templateUrl: 'games-list.component.html',

})
export class GamesListComponent  {
  /*
  The minimim required inputs of a list component using redux
  */
  @Input() games:any;
  @Input() count:number;
  @Input() page:number;
  @Input()loading:boolean;
  /*
   Emit and event when the user clicks on another page
  */
  @Output() onPageChanged = new EventEmitter<number>();

  constructor() {
  }
}

```
**games-list.component.html**
```
<div class="container" *ngIf="games">
  <table class="table table-hover">
    <thead class="thead-inverse" >
    <tr>
      <th>Name</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let game of games" >
      <td>{{game?.name}}</td>
    </tr>
    </tbody>
  </table>
  <ngb-pagination [collectionSize]="count" [(page)]="page" (pageChange)="onPageChanged.emit($event)" [maxSize]="10" [disabled]="loading"></ngb-pagination>
</div>
```
Declare the component in the application module:

**app.module.ts**
```
import {GamesListComponent} from "./components/games-list.component";
//..
@NgModule({
  //...
  declarations: [
    //...
    GamesListComponent,
  ],
  //...
})
```

`GamesListComponent` uses the [ngbPagination](https://ng-bootstrap.github.io/#/components/pagination) component which comes in the ng-bootstrap library. The component gets the `@Input`s  to render a pagination and the `pageChange` event triggers the `onPageChanged` `@output` to emit to the container component.

Next, let's modify the container component (`AppComponent` in this case).

To have a working pagination, the container  component needs to:
1. Provide the parts of the state needed as inputs for the `GamesListComponent`
2. Have a method for handling the `onPageChanged` output.


**app.component.ts**
```

import * as games from './common/games/games.actions';
//...

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements  OnInit{
  //...
  public games$: Observable<any>;
  public gamesCount$:Observable<number>;
  public gamesPage$:Observable<number>;
  public gamesLoading$:Observable<boolean>;

  constructor(private store: Store<fromRoot.AppState>) {
    /*
    Select all the parts of the state needed for the GamesListComponent
    */
    this.games$ = store.select(fromRoot.getGamesEntities);
    this.gamesCount$ = store.select(fromRoot.getGamesCount);
    this.gamesPage$ = store.select(fromRoot.getGamesPage);
    this.gamesLoading$ = store.select(fromRoot.getGamesLoadingState);
  }
  /*
   When the component initializes, render the first page ofresults
  */
  ngOnInit() {
    this.store.dispatch(new games.LoadGamesAction(1));
  }

  //...
  onGamesPageChanged(page:number) {
    this.store.dispatch(new games.LoadGamesAction(page))
  }

}
```
Lastly, add `GamesListComponent`'s selector to `AppComponent`'s template:
**app.component.html**
```
<div id="main-content">
  <!-- ... -->
  <games-list [games]="games$ | async" [count]="gamesCount$ | async" [page]="gamesPage$ | async" [loading]="gamesLoading$ | async" (onPageChanged)="onGamesPageChanged($event)"></games-list>
</div>
```
The `async` pipe uses the latest value of the observables, watches for state changes, and passes them as inputs.

Here is how pagination works in action:

![pagination](https://raw.githubusercontent.com/pluralsight/guides/master/images/eb8c1c24-1757-48c0-885b-4d056da8c7ca.com-video-to-gif_1)


# Conclusion
These examples represent many of the use cases that you might encounter when building an Angular 2 application using Redux. In a larger sense, they provide a boilerplate for more specific use cases and hopefully give new ideas for implementing other use cases.

Does Redux do a good job in controlling the UI layout? In my opinion, it absolutely does. It may require a little bit more code to be written at times, but the benefits truly start to shine as the application's codebase grows and logic gets reused. 

Missed anything? [I have uploaded the source code with all the examples on Github.](https://github.com/hggeorgiev/ng2-redux-ui-management-recipes).

___ 
Thank you for reading this guide. I hope you found it helpful and interesting. Feel free to leave questions or feedback in the comments section below.
