


State management has been an ongoing issue in front-end frameworks. The standard MVC (Model-View-Controller) approach has proven to be inneffective for managing the application state in front-end applications. For instance,  In Angular 1.x, the logic for  managing the application state was distributed between directives, controllers and services, each level having its own logic for mutating the state.  This resulted in a highly segmented application state, which was prone to causing inconsistencies and was difficult to test.

Such issues have become more apparent and difficult to circumvent as front-end applications started to become increasigly complex and more reactive to user input. However, new practices in front-end development such as the embracing of functional programming have given birth to a new concept - [Redux](https://github.com/reactjs/redux).

Redux combines

Even though Redux has originally been brought by the React community, third-party libraries such as [ngrx/store](https://github.com/ngrx/store), combined with the built-in [rxJS](https://github.com/Reactive-Extensions/RxJS) functionality, have made Redux an equally  suitable concept for use in Angular 2 applications.

## Main concepts of Redux
#### The State
#### Actions
#### Reducers
#### Effects


### Should I switch to Redux?


### Putting it into practice - build your finance planner



