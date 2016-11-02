
State management has been an ongoing issue in front-end frameworks. The standard MVC (Model-View-Controller) approach has proven to be inneffective for managing the application state in front-end applications. For instance,  In Angular 1.x, the logic for  managing the application state was distributed between directives, controllers and services, each level having its own logic for mutating the state.  This resulted in a highly segmented application state, which was prone to causing inconsistencies and was difficult to test.

Such issues have become more apparent and difficult to circumvent as front-end applications started to become increasigly complex and more reactive to user input. However, new practices in front-end development such as the embracing of functional programming have given birth to a new concept for state maangement - [Redux](https://github.com/reactjs/redux).

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

### Should I switch to Redux?


### Putting it into practice - build your finance planner



