# What we did last time

# Restructuring
  In this part, the application is going to be accomodated in a way that it can support having multiple states in its Store. To do so, the architecture of the application needs to be altered.
  
  In its current state, the application does not have a Meta-Reducer. Meta-Reducers are a map of all the reducer functions in the state tree. It contains a reference for each of the state slices and their reducers. When an action gets dispatched, the Meta-Reducer goes through the map of all reducers, looking for the one that matches the action type and calls the function.
  
 
  Another issue that causes concern is  the actions and the reducer for `operations` are staying together, but as the applicaiton grows and the actions and the reducer are becoming more complex, there's going to be a lot of code in a single file. Thus, the actions and the reducers have to be divided in different files.
  
 
  Having all these concerns in mind, here is how the directory structure will look in the new application:
  
  
  ```
+-- app
    |
    +-- reducers
    |   |
    |   +--index.ts
    +-- actions
    |
    +-- models
```

  Let's get started. First, move `operation.model.ts` to the `models` folder. This is where the models for all entities of the application will be stored from now on.
  
### "Classifying" acitons


 Next, the actions need to be overhauled so that it is known to which reducer function they belong to.
 
 In `operations.ts`, delete the action constants and create a new file in `actions/operations.ts'`
  
 Instead of making a new constant for each action, the actions will be grouped into enums.
 ```
 export const ActionTypes = {
  ADD_OPERATION: 'Add an operation',
  REMOVE_OPERATION:'Remove an operation',
  INCREMENT_OPERATION:'Increment an operation',
  DECREMENT_OPERATION: 'Decrement an operation',
};
```


Because actions will not be dispatched to a reducer directly, but instead this will be done through a Meta Reducer, @ngrx/store has provdied  the `Action` class, which lets us define custom actions and create actions as new class instances when dispatching. Expressing actions as classes also enables type checking in reducer functions.

Here's how an ``ADD_OPERATION`` action would look like:

```
export class AddOperationAction implements Action {
  type = ActionTypes.ADD_OPERATION;

  constructor(public payload:Operation) { }
}

```

The action `type` is given by the `ActionTypes` enum that was previously defined and the payload of the action in the constructor fo the class.


Lastly, a type alias for all actions in order to be later used in teh reducers.

```
export type Actions
  = AddOperationAction |
  RemoveOperationAction |
  IncrementOperationAction |
  DecrementOperationAction
  
```

 
Here is how the full code for `operations` actions:


 ```
 // app/actions/operations.ts
 
import { Action } from '@ngrx/store';
import {Operation} from "../models/operation.model";


export const ActionTypes = {
  ADD_OPERATION: 'Add an operation',
  REMOVE_OPERATION:'Remove an operation',
  INCREMENT_OPERATION:'Increment an operation',
  DECREMENT_OPERATION: 'Decrement an operation',
};




export class AddOperationAction implements Action {
  type = ActionTypes.ADD_OPERATION;

  constructor(public payload:Operation) { }
}

export class RemoveOperationAction implements Action {
  type = ActionTypes.REMOVE_OPERATION;

  constructor(public payload:Operation) { }
}

export class IncrementOperationAction implements Action {
  type = ActionTypes.INCREMENT_OPERATION;

  constructor(public payload:Operation) { }
}

export class DecrementOperationAction implements Action {
  type = ActionTypes.DECREMENT_OPERATION;

  constructor(public payload:Operation) { }
}


export type Actions
  = AddOperationAction |
  RemoveOperationAction |
  IncrementOperationAction |
  DecrementOperationAction
```
#### The new way of dispatching actions
 With the new implementations of actions, comes a new way of dispatching them. 
 
 **Before**
 ```
 //app.component.ts
 
     this._store.dispatch({type: ADD_OPERATION , payload: {
      id: ++ this.id,//simulating ID increments
      reason: operation.reason,
      amount: operation.amount
    }});
```
 
 Instead of directly sending an action with its type and payload, now this will be handled by the operation's class. Now that the action is a class, the dispatching is done by creating a new class member and putting the payload in the class constructor:
 
**After**
```
//Import the action classes for operations
import * as operations from "./common/actions/operations"


this._store.dispatch(new operations.AddOperationAction({
    id: ++ this.id,//simulating ID increments
  reason: operation.reason,
  amount: operation.amount
})
```

### Adapting the reducer function
 There are three problems with the current reducer: 
 
**1.**The type of the state (array) is too simplistic. Instead, it will be replaced with an object which contains the array and add it as an interface. This allows for the state to be more extendable in the future, when new features are implemented and there  is more data that has to be tracked.
 
```
export interface State {
  entities:Array<Operation>
};

```
 
**2.**The `operationsReducer` is not a function, but a constant of type `ActionReducer` to which has a reducer function as a value. This was done because the constant was then directly passed to `provideStore`, without being referenced in a Meta reducer.
Now that a Meta Reducer going to be implemented,  `operationsReducer` will be renamed simply to `reducer` and converted to a function with a return type of `State`.
 
**Before**

```
export const operationsReducer: ActionReducer = (state = initialState, action: operations.Actions) => { ... }
```
**After**

```
export function reducer(state = initialState, action: operations.Actions): State  {...} 
```

**3.**The new action types are not implemented. To implement them, we'll import `actions/operations` and use `ActionTypes` from it in the case statements.
 
**Before**
```
  switch (action.type) {
    case ADD_OPERATION:
      const operation:Operation = action.payload;
      return [ ...state, operation ];
  }
```
**After**
```
  switch (action.type) {
    case operations.ActionTypes.ADD_OPERATION: {
      const operation: Operation = action.payload;
      return {
        entities: [...state.entities, operation]
      };

    }
    //... rest of the cases
```   
 
 
 Here is how the new `operations` reducer looks like after all the changes have been applied:
 
 ```
import '@ngrx/core/add/operator/select';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/let';
import { Observable } from 'rxjs/Observable';
import * as operations from '../actions/operations';
import {Operation} from "../models/operation.model";


/*
  From a simple array ( [] ),
  the state becomes a object where the array is contained
  withing the entities property
 */
export interface State {
  entities:Array<Operation>
};

const initialState: State = {  entities: []};

/* 
 Instead of using a constant of type ActionReducer, the
 function is directly exported 
 */
export function reducer(state = initialState, action: operations.Actions): State {
  switch (action.type) {
    case operations.ActionTypes.ADD_OPERATION: {
      const operation: Operation = action.payload;
      /*
      Because the state is now an object instead of an array,
      the return statements of the reducer have to be adapted.
      */
      return {
        entities: [...state.entities, operation]
      };

    }

    case operations.ActionTypes.INCREMENT_OPERATION: {
      const operation = ++action.payload.amount;
      return Object.assign({}, state, {
        entities: state.entities.map(item => item.id === action.payload.id ? Object.assign({}, item, operation) : item)
      });
    }

    case operations.ActionTypes.DECREMENT_OPERATION: {
      const operation = --action.payload.amount;
      return Object.assign({}, state, {
         entities: state.entities.map(item => item.id === action.payload.id ? Object.assign({}, item, operation) : item)
      });
    }

    case operations.ActionTypes.REMOVE_OPERATION: {

      return Object.assign({}, state, {
        entities: state.entities.filter(operation => operation.id !== action.payload.id)
      })

    }
    default:
      return state;
  }

};

 ```
 
From the code we have we can conclude that a module of the state has to export:
 1. A reducer function 
 2. An interface that describes how the state looks like.
 


### Creating a Meta Reducer

With the actions and the reducer adapted to the new standards, it is time to implement the Meta reducer. If each of the reducer modules represent table in a database, the meta reducer `State` interface represents the database schema and the meta `reducer` function itself represents the database itself

. To have a clearer idea what happens behind the scenes in a Meta Reducer, we need to first look into the implementation of `combineReducers`.

 
#### combineReducers 
```
const combineReducers = reducers => (state = {}, action) => {
  return Object.keys(reducers).reduce((nextState, key) => {
    nextState[key] = reducers[key](state[key], action);
    return nextState;
  }, {});
};
```
`combineReducers` is a function that takes an object with all the reducer functions as property values and extracts its keys. Then it uses [Array.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) to accumulate the return value of each of the reducer functions into a state tree and heassign it to the key the reducer corresponds to. In the end, the state tree (Store) is returned - an object which contains the key-value pairs of the reducers and the states they returned.
 
#### Implementation

 In your `app/reducers` folder, create a new file named `index.ts`. In it, paste the following snippet:
 
```
import {combineReducers, ActionReducer} from '@ngrx/store';
import {Observable} from "rxjs";
import {compose} from "@ngrx/core";


/*
 Import each module of your state. This way, you can access
 its reducer function and state interface as a property.
*/
import * as fromOperations from '../reducers/operations';



/*
 The top-level interface of the state is simply a map of all the inner states.
 */
export interface State {
  operations: fromOperations.State;
}

/* The reducers variable represents the map of all the reducer function that is used in the Meta Reducer */
const reducers = {
  operations: fromOperations.reducer,
};


/* Using combineReducers to create the Meta Reducer and export it from the module. The exported Meta Reducer will be used as an argument in provideStore() in the application's root module. 
*/

const combinedReducer: ActionReducer<State> = combineReducers(reducers);

export function reducer(state: any, action: any) {
    return combinedReducer(state, action);
}
```
The final step of the implementation is to put the  Meta `reducer` as an argument in `provideStore()`
```
import {reducer} from "./common/reducers/index";


@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
   //...
  ],
  imports: [ 
    /*
     Put the reducer as an argument in provideStore 
    */
    StoreModule.provideStore(reducer),

  ],
})
export class AppModule {
  constructor() {}
}
```

# Getting state slices
 The new architecture of the application requires a different way for accessing the state slices. 
 
 Previously, we could simply access `operations` using the following snippet:
 
 ```
 //app.component.ts 
  import {State, Store} from "@ngrx/store";
  
  export class AppComponent {

  //....

  constructor(private _store: Store<State>) {
    this.operations = _store.select('operations')
  }

 ```
 
**However, this isn't a good idea now that there's a state tree and the `operations` state is an object that could have multiple properties.** Additionally, this doesn't allow combining states in a particlar manner. There needs to be a set of designated functions whose role is to return certain parts of the state tree.

#### Accessing parts of the state tree.
Accessing the state slices is not going to be done directly from the application's components. Instead, the `select`-ing of the state will be done in the `app/reducers` modules. 

To To illustrate how it works,let's first select `operations` from the state tree:


```
 //app/reducers/index.ts
 
 export function getOperations(state$: Observable<State>) {
   return state$.select(state => state.operations);
}

```
  
  
#### Accessing the state properties

Using the last function, only the `operations` state of the application is accessed, and it contains this:
```js
 {
    entities: [ //...an array of operations
 }
```

 That's a problem, because we need a particlar property of the `operations` state - the `entities` array. Here is how it's done:
 
 
 **First**, in `oprations.ts`, export a function which accesses the `entities` property of the `operations` state:
 
```
  
// app/reducers/operations.ts

/*

 Get the entities of the operations state object. This function will be
 imported into the file for the Meta Reducer, where it will
 be composed together with a function that gets the state of 
 the  operations state object out of the application state.
*/

export function getEntities(state$: Observable<State>) {
return state$.select(s => s.entities);
}

```

**Second**, compose `getOperations` and `getEntities`. *Function composition* is one of the building blocks of functional programming. It executes a set of functions, putting the returned value of the first function an an argument for the second function. *Remember that compose applies the result from right to left*.

```
// app/reducers/index.ts

export const getEntities = compose(fromOperations.getEntities, getOperations);
```

In this case, `getOperations` first accesses the `operations` state from the state tree and then `getEntities` gets the `entities` property of  the operations state.

Let's apply this functionality in `app.component.ts`:

```js
//app.component.ts

/*
 In order to access the application state, reference the reducers folder again,
 accessing all the exported members from it though index.ts
 */
import * as fromRoot from './common/reducers';

export class AppComponent {

  //...
  
  constructor(private _store: Store<fromRoot.State>) {
    this.operations = _store.let(fromRoot.getEntities)
  }

```

`_store.let` executes `getEntities` and returns its value.



Here is how the new `AppComponent` looks like with the new state selection and action dispatching implemented:

```js
import { Component } from '@angular/core';
import {State, Store} from "@ngrx/store";
import {Operation} from "./common/models/operation.model";
import * as operations from "./common/actions/operations"
import * as fromRoot from './common/reducers';

@Component({
  selector: 'app-root',
  template: `
      <div class="container">
           
            <new-operation (addOperation)="addOperation($event)"></new-operation>
            <operations-list [operations]="operations| async"  
            (deleteOperation)="deleteOperation($event)"
            (incrementOperation)="incrementOperation($event)"
            (decrementOperation)="decrementOperation($event)"></operations-list>
      </div>
`
})
export class AppComponent {

  public id:number = 0 ; //simulating IDs
  public operations:Observable<Operation[]>;


  constructor(private _store: Store<fromRoot.State>) {
    this.operations = this._store.let(fromRoot.getEntities)

  }

  addOperation(operation) {
    this._store.dispatch(new operations.AddOperationAction({
        id: ++ this.id,//simulating ID increments
      reason: operation.reason,
      amount: operation.amount
    })
    );
  }

  incrementOperation(operation){
    this._store.dispatch(new operations.IncrementOperationAction(operation))
  }

  decrementOperation(operation) {
    this._store.dispatch(new operations.DecrementOperationAction(operation))
  }


  deleteOperation(operation) {
    this._store.dispatch(new operations.RemoveOperationAction(operation))
  }

}

```
# Adding a new state
# Effects

### Install ngrx/effects
### make an effects directory
### make a service
### install money.js

