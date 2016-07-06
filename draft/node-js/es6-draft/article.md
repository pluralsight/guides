# 10 Reasons yo should start using ES6.

A couple of weeks ago I had an interview and the owner of the company asked me: "Are you using ES6?". I said no. And he said "Why not?". I couldn't find a good reason not to use it, and lots of reasons to do it. It doesn't matter if you are just starting out or have been developing for 10 years now. if you are developing for the web, you are using JavaScript and you should move to ES6. The new version is supported by the latest versions of all browsers, so download the latest version of your favorite browser and let's start coding!

The only downside of ES6 is that it is only supported by the newer versions of major browsers support. Luckily, there is Babel to help you out translate the new features to older JS versions that are supported by all browsers. 

### Getting Started. 
Install Babel on your project by doing `npm install --save-dev babel-cli`
Now, running `babel code/index.js -d build/` will make a `build/index.js` directory. Babel works with plugins and presets to do all its magic. For using ES6, we need to install the dependencies. Run  `npm install --save-dev babel-preset-es2015 babel-preset-stage-0`

Now, run `babel --presets es2015,stage-0 code/index.js -o build/app.js`. This will transpile your code to build/app.js that you can now launch with node `node build/app.js`



# 1. New variable declarators. 

## Let
`let` is the new, improved `var`. It fixes some things that may cause bugs if you are not aware of them. You can completely replace `var` with `let` and `const`.

### Block scope: 
Variables declared using var "leak out" of their scopes. Let's say:

```
if(myCat==="awesomefluff") {
    var awesome = true;
}
console.log(awesome); // true, leaked out var!

```
Let variables are block-scoped. Meaning they will only be available inside the {}, which is how most of the languages work. Trying to use them outside the block will cause an error: `Uncaught ReferenceError: awesome is not defined`

### Duplicates
Variables declared using `let` cannot be re-declared. This helps prevent errors and overriding variables without noticing. Again, this is how most other object oriented languages work. For example:
```
let awesome = true;
let awesome = false; // SyntaxError: Identifier 'awesome' has already been declared
```

## Const
`const` means that an identifier can't be reassigned. Trying to reassign a constant will throw an error. Keep in ming this is NOT the same as immutability. Let's say:

``` 
const myCats = ["awesomefluff", "mrBillGates"];
myCats = ["otherUncoolCat"]; //throws a Type error, you cannot reassign a const. 
```

However, you can modify the objects my const references to. So, if I buy a new cat:
``` 
myCats.push("stephenHawking"); // This is ok!
```


# 2. Arrow functions
One of the best parts of JavaScript is that it can be used for functional programming. We often do things such as .map .reduce with nameless functions. They are used only once inside this methods. Declaring them with `function` and `return` quickly becomes too verbose. Let's see:

```
// Regular function
function(x){
    return x + 5;
}

// ES6 Arrow 
x => x + 5;
```

```
// Regular function
function(x, y){
     var z = 5;
    return (x * y) + z;
}

// ES6 Arrow with params and multiple statements
(x, y) => {
    var z = 5;
    return (x * y) + z;
}
```

Using map and reduce:
```
var ages = [{name:"Travis", age:22}, {name:"Mark", age:23}];

// Regular 
var sumOfAges = 0;
for(var i = 0; i < sumOfAges.length; i++) {
    var age = sumOfAges[i].age,
    sumOfAges += age;
}

// ES5 way
var sumOfAges = ages
    .map(function(x) { return x.age; })
    .reduce(function(a, b) { return a + b; });

// ES6 way
let sumOfAges = ages.map(x => x.age).reduce((a, b) => a + b);
```










Read more at http://tutorials.pluralsight.com/front-end-javascript/getting-started-with-ecmascript6#vGqB99BOjBJmku6K.99