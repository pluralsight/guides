Redux! Redux!! Redux!!! What in the world is Redux and why do I need it? I asked myself this question when I started learning how to build single page apps (SPA) to include rich interaction on my apps. SPA has the ability to re-render different parts of the UI without requiring server roundtrip. This is achieved by separating the different data which represent the state of the application, from the presentation of these data.
The **view** layer renders a representation of these data to the UI. A view can be made up of different components. As an example, consider an online store with a product listing page. The page could contain components that represent the different products and their prices, a visual count of the total items in the cart, and a component to suggest similar products to purchased items.
The **m****odel** layer contains data to be rendered by the view layer. Each component in the view are independent of each other, each rendering a predictable set of UI elements for the given data, but multiple components can share the same data. When there is a change in the model, the view re-renders and updates the component affected by the model update.

## The Problem

The application state can be stored in random objects in-memory. It's also possible to keep some state in the DOM. But having the state scattered around can easily lead to unmanageable code. It gets hard to debug. If multiple views or components share similar data, it's possible to have that data stored in a different memory location, and the view components will not be in sync with each other.
With a separation of views from models, data gets passed from the model to the view. If there are changes based on user interactions, this will update the model and this model update could possibly trigger an update to another model and also update another view component(s) which can also trigger an update to a model.
One of the known issues with this unpredictable flow of data was the notification bug on Facebook. When you're logged in to Facebook, you see a notification for new messages. When you read it, the notification clears. After some interactions on the site, the notification comes up again, then you check and there are no new messages and the notification clears. When you interact more with the app, the notification comes back again and this goes on in a cycle.

## The Aim

It is easy to add complexity to the code if the state isn't managed properly. Therefore, it is better to have one place where the data lives, particularly when the same data has to be shown in multiple places in the view. With a haphazard flow of data, it becomes hard to reason about state changes and predict the possible outcome of a state change.

## The solution: Uni-directional data flow and single source of truth

It is easy to add complexity if the state isn't managed properly. Therefore, it is better to have one place where the data lives, particularly when the same data has to be shown in multiple places in the view. View components should read data from this single source and not keep their own version of the same state separately. Hence, the need for a **single source of truth**.
At Facebook they wanted an easier way to predict state changes and so came up with a pattern called **Flux**. Flux is a data layer pattern for managing the flow of data. It stipulates that data should only flow in one direction, with application state contained in one location (the source of truth), and the logic to modify the state just in one place.

**Flux**

![Flux](https://d2mxuefqeaa7sj.cloudfront.net/s_63B4E43BA537867EAD57F5241A742310DDD5F5BB13D7F1315135B403A279EEA3_1501269221237_the-what-and-why-of-redux.png)


The diagram above describes the flow of data in flux. 

- Data flows from the **store** (source of truth) to the **view**. The view reads the data and presents it to the user, the user interacts with different view components and if they need to modify the application state, they express their intent to do so through an **action**. 
- Action captures the ways in which anything might interact with your application. It is a plain object with a "type" field and some data. The **dispatcher** is a responsible for emitting the action to the store. It doesn't contain the logic to change the state, rather, the store itself does this internally.
- You can have multiples stores, each containing data for the different application domain. The store responds to the actions relevant to the state it maintains. If it updates the state, it also notifies the views connected to that store by emitting an event. 
- The view gets the notification and retrieves the data from the store, and then re-renders. When the state needs to be updated again, it goes through the same cycle, allowing for an easy way to reason about your application and make state changes predictable.

By implementing an application architecture that allows data to only flow in one direction, you create more predictable application states. If a bug shows up, a unidirectional data flow will make it much easier to pinpoint where the error is, as data follows a strict channel.

**Redux**
There are varying implementations of this pattern. We have [Fluxxor](http://fluxxor.com/), [Flummox](https://github.com/acdlite/flummox/), [Reflux](https://github.com/spoike/refluxjs), etc, but Redux stands tall above them all. Redux took the concepts of Flux and evolved it to create a predictable state management library that allows for easy implementation of logging, hot reloading and time traveling, undo and redo, taking cues from Elm architecture and avoiding the complexity of implementing those. 
Dan Abramov, Redux creator, created it with the intention of getting better developer tools support, [hot reloading and time travel debugging](https://www.youtube.com/watch?v=xsSnOQynTHs) but still keeping the predictability that comes with Flux. Redux attempts to make state mutations predictable.
Redux, following in the footsteps of Flux, has 3 concepts:

- **Single Source of Truth**: I've mentioned the need for this. Redux has what it calls the **store**. The store is an object that contains your whole application state. The different pieces of state are stored in an object tree. This makes it easier to implement Undo/Redo. For example, we can store and track the items in a shopping cart and also the currently selected product with Redux and this can be modeled in the store as follows:

```javascript
    {
        "cartItem" : [
            {
                "productName" : "laser",
                "quantity" : 2
            },
            {
                "productName" : "shirt",
                "quantity" : 2
            }
        ],
        "selectedProduct" : {
            "productName" : "Smiggle",
            "description" : "Lorem ipsum ... ",
            "price" : "$30.04"
        }
    }
```

- **State is readonly**: The state cannot be changed directly by the view or any other process (maybe as a result of network callback or some other event). In order to change the state, you must express your intent by emitting an action. An action is a plain object describing your intent, and it contains a type property and some other data. Actions can be logged and later replayed which makes it good for debugging and testing purpose. Following our shopping cart example, we can fire an action as follows:

```javascript

    store.dispatch({
      type: 'New_CART_ITEM',
      payload: {
                   "productName" : "Samsung S4",
                   "quantity" : 2
                }
    })
```

`dispatch(action)` emits the action, and is the only way to trigger a state change. To retrieve the state tree, you call store.getState().


- **Reducer**: The reducers are responsible for figuring out what state changes need to happen and then transforming it to reflect the new changes. Reducer is a pure function that takes in the previous (the current state about to be changed) and an action, determines how to update the state based on the action type, transforms it and returns the next state (the updated state). Continuing with our shopping cart example, let us say we want to add a new item to the cart. We dispatch an action of type **NEW_CART_ITEM** and, within the reducer, we determine how to process this new change request by reading through the action type and acting accordingly. For the shopping cart, it'll be adding a new product to the cart:

```javascript

    function shoppingCart(state = [], action) {
      switch (action.type) {
        case 'New_CART_ITEM':
          return [...state, action.payload]
        default:
          return state
      }
    }
```

What we did was to return a new state which is a collection of the old cart items, in addition to the new one from the action. Rather than mutate the previous state, you should return a new state object, and this really helps with time travel debugging. There are things you should never do inside a reducer, and they are:

- Mutate its arguments.
- Perform side effects like API calls and routing transitions.
- Call non-pure functions.

## A Practical Example

To demonstrate the workings of Redux, we're going to make a simple SPA to show how we can manage data in Redux and present the data using React. 
To set up, run the following commands in the terminal:


    $ git clone git@github.com:StephenGrider/ReduxSimpleStarter.git
    $ cd ReduxSimpleStarter
    $ npm install

We've just cloned a starter template for what we'll be building in this section. It's set up react and downloaded the Redux and react-redux npm packages. We'll be building an application that allows us to take short notes as To-do items or keywords that remind of something.

Actions are plain JavaScript objects that must have a type, and reducers determine what to do based on specified action. Let's define constants to hold the different actions. Create a new file called `types.js` in `./src/actions` with the following content:

```
    export const FETCH = 'FETCH';
    export const CREATE = 'CREATE';
    export const DELETE = 'DELETE';
```

Next, we need to define actions and dispatch them when needed. Action creators are functions that help create actions, and the result is passed to `dispatch()`. Edit the `index.js` file in the actions folder with the following content:

```javascript

    import { FETCH, DELETE, CREATE } from './types';
    
    export function fetchItems() {
      return {
        type: FETCH
      }
    }
    
    export function createItem(item) {
      let itemtoAdd = {
        [Math.floor(Math.random() * 20)]: item
      };
    
      return {
        type: CREATE,
        payload: itemtoAdd
      }
    }
    
    export function deleteItem(key) {
      return {
        type: DELETE,
        payload: key
      }
    }
```

We defined 3 actions to create, delete and retrieve items from the store. Next, we need to create a reducer. `Math.floor(Math.random() * 20` is used to assign a unique key to the new item being added. This isn't optimal but we'll use it here just for the sake of this demo. Add a new file in the reducer directory called `item-reducer.js`:

```javascript

    import _ from 'lodash';
    import { FETCH, DELETE, CREATE } from '../actions/types';
    
    export default function(state = {}, action) {
      switch (action.type) {
        case FETCH:
          return state;
        case CREATE:
          return { ...state, ...action.payload };
        case DELETE:
          return _.omit(state, action.payload);
      }
    
      return state;
    }
```

Having defined a reducer, we need to connect it to our application using the **combineReducer()** function. Inside the reducer folder, open and edit the file `index.js`:

```javascript

    import { combineReducers } from 'redux';
    import ItemReducer from './item-reducer';
    
    const rootReducer = combineReducers({
      items: ItemReducer
    });
    
    export default rootReducer;
```

We pass the reducer we created to the combinedReducer function, where the key is the piece of state that reducer is responsible for. Remember, reducers are pure functions that return a piece of the application state. For a larger application, we could have different reducers each for a specific application domain. With the **combineReducers** function, we're telling Redux how to create our application state, thus, thinking and designing how to model your application state in Redux is something you should do beforehand.
With Redux setup on how to manage our state, the next thing is to connect the View (which is managed by React) to Redux. Create a new file `item.js` inside the **components** directory. This will be a smart component because it knows how to interact with Redux to read state and request state change. Add the content below to this file:

```javascript
    import React, { Component } from 'react';
    import { connect } from 'react-redux';
    import * as actions from '../actions';
    
    class Item extends Component {
      handleClick() {
        this.props.deleteItem(this.props.id);
      }
    
      render() {
        return (
          <li className="list-group-item">
            {this.props.item}
            <button
              onClick={this.handleClick.bind(this)}
              className="btn btn-danger right">
              Delete
            </button>
          </li>
        );
      }
    }
    
    export default connect(null, actions)(Item);
```

This component displays an item and allows us to delete it. The `connect()` function takes the React component in its dumb state (i.e has no knowledge of Redux nor how to interact with it) and produces a smart component, connecting the action creators to the component such that if an action creator is called, the returned action is dispatched to the reducers. 
We will also make a second smart component that will render the previous component as a list of items and also allow us to add new items. Update the file `app.js` inside the components folder with the content below:

```javascript
    import _ from 'lodash';
    import React, { Component } from 'react';
    import { connect } from 'react-redux';
    import * as actions from '../actions';
    import Item from './item';
    
    class App extends Component {
      state = { item: '' };
    
      componentWillMount() {
        this.props.fetchItems();
      }
    
      handleInputChange(event) {
        this.setState({ item: event.target.value });
      }
    
      handleFormSubmit(event) {
        event.preventDefault();
    
        this.props.createItem(this.state.item, Math.floor(Math.random() * 20))
      }
    
      renderItems() {
        return _.map(this.props.items, (item, key) => {
          return <Item key={key} item={item} id={key} />
        });
      }
    
      render() {
        return (
          <div>
            <h4>Add Item</h4>
            <form onSubmit={this.handleFormSubmit.bind(this)} className="form-inline">
              <div className="form-group">
                <input
                  className="form-control"
                  placeholder="Add Item"
                  value={this.state.item}
                  onChange={this.handleInputChange.bind(this)} />
                <button action="submit" className="btn btn-primary">Add</button>
              </div>
            </form>
            <ul className="list-group">
              {this.renderItems()}
            </ul>
          </div>
        );
      }
    }
    
    function mapStateToProps(state) {
      return { items: state.items };
    }
    
    export default connect(mapStateToProps, actions)(App)
```

This is a smart component (or container) that calls `fetchItems()` action creator once the component is loaded. We've also used the connect function to link the application state in Redux to our React component. This is achieved using the function `mapStateToProps` which takes in the Redux state tree object as an input parameter and maps a piece of it (items) to props of the React component. This allows us to access it using `this.props.items`. The remainder of the file allows us to accept user input and add it to the application state.
Run the application using `npm start` and try adding a few items, like in the image below:

https://cdn.filestackcontent.com/uztmtifmQVfOckMNSY8Z

## Summary

Supporting rich interactions with multiple components on a page means that those components have many intermediate states. SPA has the ability to render and redraw any part of the UI without requiring a full page reload and server roundtrip. If data aren't managed properly, scattered all over the UI or put in random object in-memory, things can easily get intertwined. So, it is much better to separate the view and the models for the view. Redux does a good job of clearly defining a way to manage your data and how it changes. It is driven by 3 core principles, which are:

- A single source of truth for your application state.
- A readonly state to ensure that neither the views nor the network callbacks will ever write directly to the state.
- And transforming the state through pure functions, called reducers, for predictability and reliability.

Therefore making it a predictable state container for JavaScript application.

## Further Reading
- [Flux concepts](https://github.com/facebook/flux/tree/master/examples/flux-concepts)
- [Getting started with Redux](https://egghead.io/series/getting-started-with-redux)
- [Time travel debugging](https://www.youtube.com/watch?v=xsSnOQynTHs)

Find the source code [here](https://github.com/pmbanugo/Simple-Redux-Example).

*This was originally published on [Pusher](https://blog.pusher.com/the-what-and-why-of-redux/) under the Pusher Guest Writer program*