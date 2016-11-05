State management has been an ongoing issue in front-end frameworks. The standard MVC (Model-View-Controller) approach has proven to be inneffective for managing the application state in front-end applications. For instance,  In Angular 1.x, the logic for  managing the application state was distributed between directives, controllers and services, each level having its own logic for mutating the state.  This resulted in a highly segmented application state, which was prone to causing inconsistencies and was difficult to test.

Such issues have become more apparent and difficult to circumvent as front-end applications started to become increasigly complex and more reactive to user input. However, new practices in front-end development such as the embracing of functional programming have given birth to a new concept for state maangement - [Redux](https://github.com/reactjs/redux). With Redux, you can access the most recent values of the anywhere into your applicaiton.

Even though Redux has originally been brought by the React community, third-party libraries such as [ngrx/store](https://github.com/ngrx/store), combined with the power of [rxJS](https://github.com/Reactive-Extensions/RxJS) , have made Redux an equally suitable concept for use in Angular 2 applications.


In this guide, you are going to get acquainted with the core concepts of Redux and how they are applied in Angular 2 applications.

## Main concepts of Redux

Redux is made out of four main pieces - **the main store** , **actions** , **reducers**  each having a different role in the mutation of the application's state.  **Middlewares** are used to handle r asynchronous requests (such as API calls) and will be covered later in the guide.


#### Store
 The store combines the whole application state into a single entity, acting as the *database* for the web application. The store is broken down into different states, which represent different types of data in the application. The state is *immutable* and can be altered by explicitly defined actions. This makes debugging considerably easier, since the mutation of the state is centralized in a single point in the application.
 
#### Reducers
 If the Store is the database of the application, the reducers are the *tables* - they represent slices, or structuresin the application which are composed in a particular way. A reducer is a [pure function](https://en.wikipedia.org/wiki/Pure_function) that defines how a slice of the state is going to change when an action is being dispatched. It accepts two arguments, the previous state and an action, and returns the new state.
 
 
 ```javascript
 export interface Reducer<T> {
  (state: T, action: Action): T;
}
```
*Example*
```
export const itemsReducer: ActionReducer<number> = (state = [], action: Action) => {
  switch (action.type) {
    case ADD_ITEM: //adding an item
      const item:Item = action.payload;
      //KEEPING THE STATE IMMUTABLE:
      // We don't perform actions that alter the state such as
      // array.push, array.shift ad so on.
      //instead, we concatenate the item
      return [ ...state, item ]; 

    case REMOVE_OPERATION://removing the item
    //Again, we don't remove the item, we just filter the new state so
    //that it won't contain the item we're removing
      return state.filter(operation => {
        return operation.id !== action.payload.id;
      });
  }
}
```
#### Actions
 Actions represent payloads of information that are dispatched to the store from the application and are usually trigerred by user interaaction. Each reducer has a set of action types that define how the state should be changed. An action is composed of two parts - a type and a payload:
 
 ```
 export interface Action {
  type: string;
  payload?: any;
}
```

*Example*:
````
//making an action for adding an item
dispatch({type: ADD_ITEM, payload: {id: 1, name: 'An item' , category: 'miscellaneous'}})
````
##### Review

 When an action is dispatched, the reducer takes it and applies the payload depending on the action type, and outputs the new state.
 
 
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/79263077-e972-47c6-93dc-44e466a8e191.gif)


 
 The store encompasses the whole state, the reducers return fragments of the state, and actions are pre-defined, user-trigerred events that communicate how a given fragment of the state should change.Middlewares are used in cases the actions require asynchronous requests. The reducer takes its previous state, applies the new action to it, and returns it back. 


#### Projecting data
 
 As you already know, the Store is a tree-like structure representing the application state, that is composed of reducers, which represent different slices of the state. In the case of Angular 2's [ngrx/store]() , the store is an [observable](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html), hence making the access to the application's state reactive and providing the opportunities to mix the values of several states using RxJS's operators. 
 
```
//getting a single slice of the state
store.select('items')

//combine multipleslices
Observable.combineLatest(
  store.select('items'),
  store.select('categories'),
  (items, categories) => {
    
})
```



## Redux in practice

 Now that you know the main concepts, you're probably wondering how they tie together in an Angular 2 application. To illustrate how Redux works, we are going to build a simple financial accounting tool that will keep track of your transactions using the Redux architecture. It will feature operations which will either add or deduct money from an imaginary account. Later on we are going to add a way to see your current balance and some additional statistics.
 
 
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b3ec1192-be69-4e0a-b635-239a99bf3eef.001)

 ### Setup
 
We'll use [angular-cli](https://github.com/angular/angular-cli) to setup the project:

```bash
 ng new financials_app
 cd financials_app
```

[ngrx/store]() is a state container that is specifically built for Angular 2 applications. It is going to provide the utilities and the building blocks for the Redux architecture.

```bash
 npm install @ngrx/core @ngrx/store --save
```

*Optionally*, you can also install [Bootstrap](https://v4-alpha.getbootstrap.com/) to make the application look good and well-structured:

```
 npm install bootstrap@next
```

Add to your `angular-cli.json` the following lines:
In the `app.scripts` array, as an object property:
```
"scripts": [
  "../node_modules/jquery/dist/jquery.js",
  "../node_modules/tether/dist/js/tether.js",
  "../node_modules/bootstrap/dist/js/bootstrap.js"
],
```
In the `app.styles` array:
```
"styles": [
  "../node_modules/bootstrap/dist/css/bootstrap.css",
  "styles.css"
]
```

 ### Your first reducer
 
 The first thing to do in a Redux application is to define your store and start attaching reducers to it. For the first iteration of the application, we will add the state of financial operations.
 
 In your Angular 2 app,  make a new directory that will contain the reducers of your application:
 
 ```
   cd src/app
   mkdir common 
 ```

 First, let's make a model of our financial operation:
```javascript
//src/app/common/operation.model.ts
export class Operation {
  id:number;
  amount:number;
  reason:string;

  constructor() {}
}
```

#### Defining a reducer function
 Next, create a file that will contain the definitions of the actions and the reducer for the financial operations:
 
```
//src/app/common/operations.ts
import {ActionReducer, Action, State} from '@ngrx/store';
import {Operation} from "./operation.model";


//definitions of the actions that alter the state
export const ADD_OPERATION = 'Add an operation';
export const REMOVE_OPERATION = 'Remove an operation';
export const INCREMENT_OPERATION = 'Increment an operation';
export const DECREMENT_OPERATION = 'Decrement an operation';


//the initial state of the operations
const initialState:State = [];

//the operationsReducer - a pure function that is responsible for maintaining the 
//financial operations state of your store
export const operationsReducer: ActionReducer = (state = initialState, action: Action) => {
  switch (action.type) {
   //In Redux, you cannot mutate the state. In this case using .push(), .pop(), 
   // .shift() or .unshift() is against the convention. 
    case ADD_OPERATION://Action type 
      const operation:Operation = action.payload;//the contents of an operation
      return [ ...state, operation ];

    case INCREMENT_OPERATION: 
      const operation = ++action.payload.amount;
      return state.map(item => {
        return item.id === action.payload.id ? Object.assign({}, item, operation) : item;
      });

    case DECREMENT_OPERATION:
      const operation = --action.payload.amount;
      return state.map(item => {
        return item.id === action.payload.id ? Object.assign({}, item, operation) : item;
      });

    case REMOVE_OPERATION:
      return state.filter(operation => {
        return operation.id !== action.payload.id;
      });


    default://if the action.type is unknown, return the state
      return state;
  }

};
```
 The reducer's state is an array of financial operations. For each action that is dispatched to the reducer (or, to put it simply, every time the reducer function is called), a switch operator creates a **new state**, depending on the action type, with the changes applied. If the action type does not match any of the defined actions, the state is simply returned.
 
 Note that the state is **immutable**. Instead of changing the array of operations, we create a copy of it and apply the changes to the new copy.
 
#### Initializing the store
 The next step is to put the reducer into the store. 
 
```
// src/app.module.ts
import { AppComponent } from './app.component';
import {StoreModule} from "@ngrx/store";
import {operationsReducer} from "./common/operations";//import the reducer
import {CommonModule} from "@angular/common";


@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent
    //...
  ],
  imports: [ // import StoreModule
    //...
    StoreModule.provideStore({ operations: operationsReducer }) //provideStore accepts an object with reducers.
  ],
})
export class AppModule {
  constructor() {}
}
```
At this stage of the application, there is only one slice of the state and consequently, only one reducer. However, at later stages, we'll have multiple reducers and we'll use [combineReduceers](http://redux.js.org/docs/api/combineReducers.html) in order to provide a helper for dispatching actions. The purpose of ``combineReducers`` is to simply act as a helper for combining all the returned states from the reducers in a single entity that will represent the application store.

By putting the `operationsReducers` as an argument in `provideStore`, you can access the state of `operations` in any place of the application by importing `Store` from `@ngrx/store`. Here is how you can do it:


#### Projecting the state
```
//src/app/app.component.ts
import { Component, ViewEncapsulation } from '@angular/core';
import {Operation} from "./common/operation.model";
import {State, Store} from "@ngrx/store";
@Component({
  selector: 'app',
  template: `{{ operations | json }}`
})
export class AppComponent {

  public operations:Array<Operation>;

  constructor(private _store: Store<State>) {
    //By using observables, @ngrx/store lets you have access to the most recent state in real time
    _store.select('operations').subscribe(state => this.operations= state)
  }
}
```

By injecting the store and using `store.select()`, you can access one of the states in the application (from the reducers you put in `provideStore()`. In this case, we only have `operations`. The `store` always returns an observable, so you have to `subscribe` to it, or, if you are passing it in a child component, use the `async` pipe (an example will be given at a later stage). In the template of the component, we'll use the `json` pipe to see the state of `operations`. At this point, it is an empty array (`[]`).

### Dispatching actions