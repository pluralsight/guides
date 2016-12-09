# What we did last time

# Restructuring
  In this part, the application is going to be accomodated in a way that it can support having multiple states in its Store. To do so, the architecture of the application needs to be changed.
  
  In its current state, the main issue that the application has, is that it doesn't have a Meta-Reducer. Meta-Reducers are a map of all the reducer functions in the application. When an action gets dispatched, the Meta-Reducer goes through the map of all reducers, looking for the one that matches the action.
  
  
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
 
### Adapting the reducer function
 There are three problems with the current reducer: 
 
 1. The type of the state (array) is too simplistic. Instead, it will be replaced with an object which contains the array and add it as an interface.
 
```
export interface State {
  entities:Array<Operation>
};

```
 
 2. The `operationsReducer` is not a function, but a constant of type `ActionReducer` to which has a reducer function as a value. This was done because the constant was then directly passed to `provideStore`, without being referenced in a Meta reducer.
  Now that a Meta Reducer is used,  `operationsReducer` will be renamed simply to `reducer` and converted to a function with a return type of `State`.
 

**Before**
```
export const operationsReducer: ActionReducer = (state = initialState, action: operations.Actions) => { ... }
```
**After**
```
export function reducer(state = initialState, action: operations.Actions): State  {...} 
```

 3. The new action types are not implemented. To implement them, we'll import `actions/operations` and use `ActionTypes` from it in the case statements.
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


export function reducer(state = initialState, action: operations.Actions): State {
  switch (action.type) {
    case operations.ActionTypes.ADD_OPERATION: {
      const operation: Operation = action.payload;
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


### Implementing the Meta Reduceer
 
 ** Reducers **
 ### combineReducers and the metareducer
 
# Multiple states
- states
- reducers map

### Total change of the operations reducer
### Separating actions


### Combining states
### Changing the app.module

# Getting state slices

# Effects

# UI State