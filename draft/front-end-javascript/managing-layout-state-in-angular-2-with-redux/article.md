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

The easiest way to implement a modal in the state is to keep its name as an identifier. Since only one modal can be opened at a time in the layout (unless you're trying to do some kind of black magic), every modal can be referenced by a `modalName`.

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

### ** GIF GOES HERE ** 

# Sidebar(s)
 The most generic representation of a sidebar in an application comes down to whether the sidebar is opened or not. The state will have a property that denotes whether a sidebar is `opened` or `closed` using a boolean value. In case there are two sidebars (or more, depends on what kind of sorcery your're doing), there will be a property in the state for each. 
 
 For the user to start interacting with the sidebar, there need to be actions for opening and closing:
 
 **layout.actions.ts**
 ```
 export const LayoutActionTypes =  {
  //Left sidenav actions
  OPEN_LEFT_SIDENAV: type('[Layout] Open LeftSidenav'),
  CLOSE_LEFT_SIDENAV: type('[Layout] Close LeftSidenav'),
  //Right sidenav actions
  OPEN_RIGHT_SIDENAV: type('[Layout] Open RightSidenav'),
  CLOSE_RIGHT_SIDENAV: type('[Layout] Close RightSidenav'),
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
 As it was already discussed, the states of the sidebar will be represented with boolean variables. In this case, the left sidenav will be open by default, but there can be logic that checks if the window size is small enough to dispatch a `CloseLeftSidenavAction` to close it:
 
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


 Instead of putting the logic in each of the sidebar components, it can be combined within a structural directive which closes and opens the corresponding sidebar depening on the state of the application.
 
 ```
 $ mkdir src/app/diretives
 $ touch src/app/directives/sidebar-watch.directive.ts
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
 The directive checks to which sidebar is applied to by checking `ElementRef`'s `nativeElement`, which makes the DOM properties of the template accessible in the component.
 Once it is decided to which sidebar it is applied to, the directive checks whether the corresponding state (`LeftSidenavbarState` or `RightSidenavbarState`, respectively) is `true` or `false`. Then the directive uses jQuery to manipulate the corresponding elements in the layout. The use of jQuery to select and directly change the DOM element's style properties is optional and you can use addition and removal of classes or by using plain JavaScript.
 
 Following the same logic, there can be a directive for toggling the sidebars:
 
 ```
  $ touch src/app/directives/sidebar-toggle.directive.ts
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
 
 The directive has an `@Input sidebarToggle` which can be either `left` or `right` , depending on which sidebar the directive has to control.  Every time the use clicks on the element to which the directive is attached  to, the `@HostListener('click')` catches the event and checks the state of the sidebar of the store and dispatches the corresponding action.
 
 Next, add the directives to the root module:
 
 **app.module.ts**
 ```
import {SidebarWatchDirective} from "./directives/sidebar-watch.directive";
import {SidebarToggleDirective} from "./directives/sidebar-toggle.directive";


 @NgModule({
  declarations: [
    //...
    SidebarWatchDirective,
    SidebarToggleDirective,
    ]
    //...
})
 
 ```
To demonstrate how everything come together, let's make two sidebars:
```
$ touch src/app/components/left-sidebar.component.ts
```

**left-sidebar.component.ts**

```
import { Component } from '@angular/core';

@Component({
  selector: 'left-sidebar',
  templateUrl: 'left-sidebar.template.html',
  styleUrls: ['./sidebar.styles.css']
})
export class LeftSidebarComponent  {
  constructor() {}
}
```
```
$ touch src/app/components/left-sidebar.template.html
```
**left-sidebar.template.html**
```
<section sidebarWatch class="left-sidebar">
</section>
```

```
$ touch src/app/components/right-sidebar.component.ts
```
**right-sidebar.component.ts**
```
import { Component } from '@angular/core';

@Component({
  selector: 'right-sidebar',
  templateUrl: 'right-sidebar.template.html',
  styleUrls: ['./sidebar.styles.css']
})
export class RightSidebarComponent  {
  constructor() {}
}
```
```
$ touch src/app/components/right-sidebar.template.html
```
**right-sidebar.template.html**
```
<section sidebarWatch class="right-sidebar">
  <button class="btn btn-primary" sidebarToggle="right">Close Right Sidebar</button>
</section>
```
Usage of the `sidebarWatch` directive is quite straightforward - they just need to be put in the topmost elements of the sidebars. 

The `sidebarToggle` needs to be put in a button (although you can put it anywhere you like) and it needs to have the `left` or `right` value assigned to it.

To make the elements look and feel like sidebars, there needs to be some additional css:

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

Add the sidebars to the root module:
 **app.module.ts**
 ```
import {LeftSidebarComponent} from "./components/left-sidebar.component";
import {RightSidebarComponent} from "./components/right-sidebar.component";

 @NgModule({
  declarations: [
    //...
    RightSidebarComponent,
    LeftSidebarComponent
    ]
    //...
})
 ```

In the root component of the application, put the sidebars on top all the content in a div with a class 'main-content':
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

With this setup, the sidebars are truly container-agnostic. Any element can be made a sidebar through a directive and be toggled from any point of the layout. There is also flexibility if there's a requirement  to add additional components such as bottom bars or top bars.

# Pagination

# Accordion 

# Confirmation dialogs

# Window size

# Events

# Colors
