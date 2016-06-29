Recently I have been doing a lot of work with [Redux](http://redux.js.org/) for structuring data in a production React application. One of Redux's tenets is that the functions used to manipulate state must be pure. Essentially, a new state needs to be created and returned each time data is changed, but this state creation must not modify the original state. Enforcing this state immutability is extremely important with Redux, as all of its features are contingent on the assumption that states are immutable. 

## Importance of immutability

Working with Redux has helped me understand the importance of immutability when writing large software applications. Even if you don't use Redux, working with pure functions helps the long-term maintainability of a codebase by preventing side-effects. If functions are written with immutability in mind, each function will clearly express what it accomplishes, thus avoiding unintended interactions. Writing code in this manner makes it easier to follow the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), which encourages smaller, more specific functions.
 
Unfortunately JavaScript does not make it very easy to enforce immutability when dealing with arrays and objects. We can, however, utilize some new ES2015 (or ES6) features to make dealing with these reference variables much easier.
 
## Arrays 
The `push()` method, although convenient, cannot be used when we want to enforce immutability. Here is an example of how the `push()` method affects variables passed into a function. 

```javascript
function addTwo (arr) { 
  return arr.push(2);
} 
 
var original = []; 
var newArray = addTwo(original); 
console.log(original); // [2]
console.log(newArray); // [2] 
``` 
When passing parameters into a function, we would typically expect that function to create a copy of the parameters to use within the function body. However, since arrays are reference variables, the reference that we pass into a function will be the same reference that is used within the function body. **Therefore, when we perform operations like `push()` on that reference, it also affects the original array outside of the function.** 
 
We can get around this by using the `concat()` operator instead of `push()`. `concat()` actually returns a new array when it is called, consisting of the two arrays that are passed into it as parameters. 
 
The above example now becomes: 

```javascript
function addTwo (arr) { 
  return arr.concat(2); 
} 
 
var original = []; 
var newArray  = addTwo(original); 
console.log(original);  // []
console.log(newArray); // [2]
``` 
 
We now have a function that appends the number 2 to an array without modifying the original array. 
 
With ES2015, we can simply truncate the syntax by using the spread operator (`…`) instead of the `concat()` method.  

```javascript
// ES2015 
function addTwo (arr) { 
  return [...arr, 2]; 
} 
``` 
 
In a similar fashion, we can actually remove an element from an array using the spread operator while still enforcing immutability. In this example, `slice()` is used as a safe method that returns a shallow copy of an array. 
 
```javascript
function removeItem (arr, index) { 
  return [ 
    ...arr.slice(0, index),
    ...arr.slice(index+1) 
  ]; 
} 
 
let original = ['a', 'b', 'c', 'd']; 
let newArray = removeItem(original, 2); 
 
console.log(original); // ["a", "b", "c", "d"] 
console.log(newArray); // ["a", "b", "d"] 
``` 
 
## Objects 

Since Objects are also reference variables, we cannot pass one into a function and modify it directly without mutating the original variable. As with arrays, when using objects, we have to figure out a way to create a function that modifies the object by returning a brand new object. 

Luckily, ES2015 gives us an extremely useful method, [`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).

Say we have a todo-list application that modifies todos of the form:
```javascript
exampleTodo = {
  id: 0,
  text: 'Learn immutability',
  completed: false
};
```
and we want to create a function that toggles whether or not a todo is completed. To do this, we could simply pass in the object and modify the `completed` property directly, using `exampleTodo.completed = true`. However, as we already know, this is a mutation that we want to avoid.

Let's instead use `Object.assign()` to create a function that will toggle the `completed` property by returning a new todo. `Object.assign()` works by copying the its parameters' property values into a new object. The first parameter of the `Object.assign()` method defines the target object that we want to create, which in our case is an empty object, `{}`. All other parameters define the properties that we want to copy into this new object.

```javascript
function toggleTodo (todo) {
  // Copy todo properties into a new object and
  // overwrite the completed property with
  // a new value
  return Object.assign({}, todo, {
    completed: !todo.completed
  });
}

let newTodo = toggleTodo(exampleTodo);

console.log(exampleTodo);
// { id: 0, 
//   text: 'Learn immutability', 
//   completed: false }

console.log(newTodo);
// { id: 0, 
//   text: 'Learn immutability', 
//   completed: true}
```

Now we can successfully alter the information contained within our todos without mutating the original todo.