
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


# 3. Strings

## Template strings
This isn't as much as an innovation as it is something that should have been there a long time ago. Instead of concatenating strings, we now use template strings that insert values dynamically.

```
// ES5 way
console.log("Hello " + name + " you are user #: " + userNumber);

// ES6 way
console.log(`Hello ${name} you are user #: ${userNumber}`);
```
## Multiline Strings!
````
var roadPoem = `This is the first line, 
                this one is second,
                this is third.`
```




# 4. Classes
The difference between JavaScript and other classic Object Oriented languages such as Javais the way of inheritance. JS use prototypal inheritance rather than classes. ES6 now somulates classes using the class keyword. You may or may not like this on JS, but it is widely used on mainstream libraries like React. So, let's take a look

```
// ES5 using prototypes
function Cat(name){
    this.name=name
}
Cat.prototype.wakeUp = function (){
    "Hey " + this.name + " wake up!"
}

// ES6 classes
class Cat{
    constructor(name){
        this.name=name;
    }
    wakeUp = function (){
    "Hey " + this.name + " wake up!"
    }
}
```

# 5. Default Parameters

To get default parameters on prevous versions we had to assign them manually at the beggining of the cunction.


Remember we had to do these statements to define default parameters:
```
var catTalk = function (name, phrase) {
    var name = name || 'No One'
    var phrase = phrase || 'Hi'
    return name + ', says ' + phrase;
}
```
des
Now, in ES6 we can do:
```
var link = function(name = 'No One', phrase = 'Hi') { }
```


# 6. Modules

There us no native way of using modules in JS before ES6. To solve this problem, the communities have developed AMD, CommonJS and many other solutions for this. If you have been developing with node, you are familiar with the CommonJS syntax that can also be used on the browser with Browserify.

Now we have a native alternative in ES6. Let's take a look.

```
// config.js
module.exports = {
  port: 3000
}

// index.js
var service = require('config.js')
console.log(service.port) // 3000
```

On ES6, we import/export stuff.

```
// config.js
export const port = 3000;
export function getENVSettings(r) {
    ...
}

// index.js Option 1
// As you can see, you can export many values and import them as needed.
import {port, envSettings} from "./config";
console.log(port);

// index.js Option 2
import * as config from "./config";
config.getENVSettings();
```

I don't think node will be changing their module system anytime soon. But it's good to have options. The important thing is to write modular code. 

# 7. Assignments

In ES6, you can do many assignments at once instead of doing it line by line. Assigning a name for your cats just became simpler with array destructuring.

```
// ES6 Way
let [cat1, cat2, cat3] = ["Mark", "Travis", "Tom"];

// ES5 Way
var cat1 = catNames[0];
var cat2 = catNames[1];
```

An example of swapping values:

```
// ES6 way
[cat1, cat2] = [cat2, cat1];

// ES5 way
var tmp = cat1;
cat1 = cat2;
cat2 = tmp;
```
It works for objects too!

```
var catInfo Data = {
    name: "Mr. Catcat",
    radioactive: true
};

// ES6 way
let {name, radioactive} = catInfo;

// ES5 way
var name = catInfo.name;
var radioactive: catInfo.radioactive;
```

This is amazing because you can do this: 
```
var {username, password} = req.body
```









