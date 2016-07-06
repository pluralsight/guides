# 10 Reasons yo should start using ES6.

A couple of weeks ago I had an interview and the owner of the company asked me: "Are you using ES6?". I said no. And he said "Why not?". I couldn't find a good reason not to use it, and lots of reasons to do it. It doesn't matter if you are just starting out or have been developing for 10 years now. if you are developing for the web, you are using JavaScript and you should move to ES6. The new version is supported by the latest versions of all browsers, so download the latest version of your favorite browser and let's start coding!

The only downside of ES6 is that it is only supported by the newer versions of major browsers support. Luckily, there is Babel to help you out translate the new features to older JS versions that are supported by all browsers. 

### Getting Started. 
Install Babel on your project by doing `npm install --save-dev babel-cli`
Now, running `babel code/index.js -d build/` will make a `build/index.js` directory. Babel works with plugins and presets to do all its magic. For using ES6, we need to install the dependencies. Run  `npm install --save-dev babel-preset-es2015 babel-preset-stage-0`

Now, run `babel --presets es2015,stage-0 code/index.js -o build/app.js`. This will transpile your code to build/app.js that you can now launch with node `node build/app.js`

# Let's go Crazy.

## New variable declarators. 

`let` is the new, improved `var`. It fixes some things that may cause bugs if you are not aware of them. You can completely replace `var` with `let` and `const`.

Block scope: Variables declared using var "leak out" of their scopes. Let's say:

```
if(myCat==="awesomefluff") {
    var awesome = true;
}
console.log(awesome); // true, leaked out var!

```
Let variables are block-scoped. Meaning they will only be available inside the {}, which is how most of the languages work. Trying to use them outside the block will cause an error: `Uncaught ReferenceError: awesome is not defined`


Duplicate let declarations are forbidden
let is designed to catch potential assignment mistakes. While duplicate var declarations will behave like normal reassignment, duplicate let declarations are not allowed to prevent the common mistake of erroneous reassignment.

var x = 1;
var x = 2; // x equals 2

let x = 1;
let x = 2; // SyntaxError: Identifier 'x' has already been declared
let variables rebound in each loop iteration
Here is a common error that occurs when you have a function defined inside of a loop using var.

for(var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, 10);
};
// logs 5 5 5 5 5
This code will log the number 5 five times in a row, because the value of i will be 5 before the first time console.log is called. When we use let instead, the i inside of the function will correspond to the value on that particular iteration of the for-loop.

for(let i = 0; i < 5; i++) {
    setTimeout(() => {
        console.log(i);
    }, 10);
};
// logs 0 1 2 3 4













`const` declares constants and will throw errors when overwritten.


const is used for constant declarations, and let is used for variable declarations.

If you try to reassign to a constant, the compiler will throw an error:

const one = 1;
one = 2; // SyntaxError: "one" is read-only


Read more at http://tutorials.pluralsight.com/front-end-javascript/getting-started-with-ecmascript6#vGqB99BOjBJmku6K.99