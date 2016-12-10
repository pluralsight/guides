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


### Implementing the Meta Reducer

With the actions and the reducer adapted to the new standards, it is time to implement the Meta reducer. This is done by using `combineReducers`.


 
#### combineReducers 
```
const combineReducers = reducers => (state = {}, action) => {
  return Object.keys(reducers).reduce((nextState, key) => {
    nextState[key] = reducers[key](state[key], action);
    return nextState;
  }, {});
};
```
`CombineReducers` takes an object with all the reducer functions as property values and extracts its keys. Then it uses [Array.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) to accumulate the return value of each of the reducer functions into a state tree and heassign it to the key the reducer corresponds to. In the end, the state tree (Store) is returned - an object which contains the key-value pairs of the reducers and the states they returned.
 
#### Multiple states
- states
- reducers map





#### Changing the app.module

# Adding a new state
# Getting state slices

# Effects

