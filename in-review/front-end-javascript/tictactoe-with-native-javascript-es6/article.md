# Introduction
We will learn how to create a ES6 pure JavaScript game of Tic Tac Toe. To do this, we will create Components, Libraries and a Module.

Before we get started:

- Make sure you moderately understand DOM Level 1~2 
- Be certain to have some experience with Javascript ECMAScript 5 (ES5) under your belt
- I will assume that you know a little bit about ECMAScript 6 

# Setting up the environment
Since the purpose of this article is to build a pure JS implementation of Tic Tac Toe, complete with minimum HTML
necessary, we will need [Browserify](http://browserify.org/), [Babelify](https://babeljs.io/docs/setup/#installation),
[NodeJs](https://nodejs.org), and an IDE of your choosing with access to a Terminal/shell.

Install NodeJS by downloading it directly from the [official website](https://nodejs.org/en/download/).

If you don't have a Terminal, you can download [git-scm](https://git-scm.com/downloads) where you can download and set up a Git Bash terminal.

# Creating the Workspace
First, we need a folder for our project. And since we are going to have Browserify bundle our code we might as well make `src` and `dist` folders within the project folder:

```logtalk
$> mkdir tic-tac-toe
$> cd tic-tac-toe
$> mkdir src
$> cd src
$> mkdir lib modules components
$> cd ..
$> mkdir dist
```

Now we are ready to initiate our packaging, issue `npm init` and answer the questions.
Since we will be using git further on, lets go ahead and create a `.gitignore` file at the root of the project (tic-tac-toe)
and have `node_modules/` as the first line (so we can ignore our dependencies when we send the project to git).

#### Naming and export
Each file should have a good description of what it does. If your file name can be read as performing various functions, you should probably refactor that functionality into two separate files. This way we can keep growing our game simple.

The only thing that can be exported from files are Classes, Functions and Objects (if we need to use some option Model,
for instance. Though those will not exist in this Article.)

The export naming should be the same as the name of the file;

| name of the file | name of the export |
|:---------------- |:------------------ |
| the-thing.js     | TheThing           |
| word.js          | Word               |


## Creating new functionality

#### Organization hierarchy

Understand which functionality you want to create, and where it belongs.

pex: If you want to change the value of a element `textContent`, that functionality should be on the __Component__ level;

Whereas if you wanted to count how many elements there are of `selector` that should be created as a __Module__ which would then be imported where that functionality is needed.

#### Code style

This time, I refrained from my habit of using a [JavaScript Linter](http://www.javascriptlint.com). However, some practices that I adhered to:
- use strict
- return early
- mind the semi-colon
- spaces not tabs

Accompanied with the DRY way, these are more than enough for a nice and understandable read of the code. I also like to code as descriptively as possible. This way methods remain small and tidy and reading code becomes a walk in the park.

# Installing and setting dependencies
Now that we have our workspace all tidied up, and node_modules ignored, we can set up our dependencies.
We spoke the names of two "ify" things, Browserify and Babelify: These are the node modules that will help translate our ES6 code into browser-readable ES5.

Why? Because some browsers don't support the ES6 moduling yet, and that's an intricate part of our program design. The alternative is to write the whole game into a single-file - which no one in his right mind would care to look more than once for the first 20 lines or so. To avoid this, we use Browserify.

Browserify understands the ES6 module, takes care of bundling everything into a neat file, and then gives it to
Babelify which translates our ES6 code with the latest preset (so we can use all the goodies, though we shan't abuse them in this first article).

```logtalk
$> npm install --save-dev browserify babelify babel-preset-latest
```

Then, edit your `package.json` to contain the following reference to the browserify module

```json
{
    "browserify": {
        "transform": [ "babelify" ]
    }
}
```

And so we don't have to type our fingers off every time we need test our application, add the following scripts to the
`scripts` property of the package.json

```json
{
"scripts": {
    "prebundle": "cp src/index.html dist/index.html",
    "bundle": "browserify src/game-start.js -t babelify --outfile dist/bundle.js"
  }
}
```

Now, in order for Babelify to know which preset to use, create a new file named `.babelrc` with:
```json
{ "presets": ["latest"] }
```

#### Sanity check!
Let's go over what our code actually does. The `bundle` script creates the bundled file of every file that's required by any file that's required by `src/game-start.js`. The `prebundle` script copies the `src/index.html` file into `dist/index.html`

Some would say a tool like [Gulp or Grunt](https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4) would be perfect here, but these are likely overkill. npm scripts are more than enough to automate what we need in this circumstance. However, with larger projects, I would suggest freestyling to figure out what works best. 

## Project Design
### Modules, Libraries and Components
Modules, libraries, and components are nothing more than Classes, which have different functions are distinguishable from each other. 

#### Modules
Modules glue Libraries and Components together. These are foundational to JavaScript development, and are used in everything.

#### Libraries
Libraries are single-action classes. For instance, our GameEngine is a library because it is only responsible for holding turn information and occupying a zone.

#### Components
While not as advanced as WebComponents, our Components are the class representation of an HTML element in the page. This is achieved by assigning a `element` property to the class on `constructor` which itself is a `document.createElement()` so that it can then be attached by Modules.

### ES6 Classes
The [MDN](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/) has much to say about Classes, but you can think of them as the old revealing module pattern:
```javascript
var TheThing = function() {
 function privateFn() { /** ... */ }
    return { name: "The Thing"};
};
```

However, instead of the `function` tag, we have the `Class` statement. This comes with all the goodies that comes from having classes: `extends`, `get`, `set` to name a few (but then you have `static` and the sorts).

As a Class, "TheThing" would be written as:
```javascript
class TheThing {
    constructor() {
        this.name = "TheThing";
        function privateFn() { /** ... */ }
    }
}
```

In this case, you wouldn't be able to call `privateFn` from outside the constructor. People have submitted proposals for private methods but these have not been implemented yet. (We could use the latest preset sugar coating strawman `::` but often times it doesn't feel right. As such, we either prename the private functions with a `_` or we simply don't write private functions.)

## Coding Tic Tac Toe
### Basics of the game: What do we know about Tic-Tac-Toe?

1) We know the field of the game is usually a 3x3 grid but that [4x4 grids are also possible.](http://all-r-math.blogspot.com/2012/11/4x4-tic-tac-toe-ii.html)
2) The game is played by two players, denoted by either "X" or "O". Let's keep this aspect of the game constant.
3) A winner is declared once he/she fills a row, column, or main diagonal with his/her symbol.
4) The game can either result in a win/loss or a tie.

#### Grid design

Starting from the top, we now know we need to create some sort of map. Sure, we could just create a simple Array object composed of three arrays containing three arrays (for a total of 9 squares). However, we can already imagine the ungodly mess that that will create. Not to mention the issue of resizing our grid for subsequent, interesting implementations.

Instead, we should create a library that will take desired grid size as an argument and iterate until we have a representational square in an array. Since we will create a neat array, why not insert
an object as the GameSlot - that way we can track it for occupation and symbol without much fuss.

Q: "Why not create a simple Object with indexes to Row and Column?"

A: Think about all the looping you'll have to do just to figure out a single horizontal line. If we have a simple object, we will have to loop through ALL the entries just to find 3 matching ones - that's counter-intuitive, and while not resource-intensive, (because it's only a 3x3) one could argue best practices take precedence here.

As a result, let's create separate square GameSlots that will contain symbols and allow us to check filled nodes.

### Map Library

Create a new file, `map.js` (under `lib/`) which will have the following content ([see it on repl.it](https://repl.it/FRr0/1))

```javascript
/** Create a representation of a square grid using Arrays that will match to our "game map" */
export function createSquare(height) {
    const rows = height || 3; /** use the provided argument or default to size 3x3 */
    const columns = height || 3;
    const field = [];
    for (let x = 0; x < rows; x++) {
        let row = [];
        for (let y = 0; y < columns; y++) {
            /** we set the row/column information inside the slot so we can return an array of slots on win conditions
             * that way we won't to make any special changes to the slot object in the future. */
            let slot = {occupied: false, symbol: '', row: x, column: y};
            row.push(slot);
        }
        field.push(row);
    }
    return field;
}
```

### GameHud
Now that we have a map we can start building up what our user will see. Before getting into the code, we need to design our Components. Let's assess what kind of Components we need, so we don't end up repeating functionality.

### Game visuals

What do we already know about how Tic-Tac-Toe looks to a user?

1) We know that players write their symbols on the game field.
2) We know that lines are used to cross filled columns, rows, or main diagonals to show that the game has ended.
3) We know whose turn it is.

From these three points we can take that we will need some sort "write value" functionality in our Components and a sort of List to be able to get relevant information from the element. Since we have two functionalities, we will create two files to split up our tasks.

Start from a base component. Since each component has to have a DOM representation, we can say that every Component is, in the end, "the same."


#### Base Components

Lets go ahead into our `lib/` folder and create the `simple-component.js` file:

```javascript
export class SimpleComponent {
    /**
     * This class generates a element property containing a NodeElement to be appended into the DOM. This class should be
     * the base of every other component we will create and therefore should also contain its removal from dom.
     * @param selector {String} the selector of the element to be created
     */
    constructor(selector) {
        if (!selector) throw Error('a SimpleComponent must be composed of a selector');
        this.selector = selector.toString();
        this.element = document.createElement(this.selector);
    }

    /** call this when you want to remove the element from the DOM */
    destroy() {
        document.body.removeChild(this.element);
    }
}
```

We are now ready to extend this class and make our "write value functionality", by creating another file inside the same
folder, `writable-component.js`. This file will extend `SimpleComponent` and will take in the same `selector` argument.

The constructor of this class is composed of `super(selector)` so we can pass down the argument that will be provided to
create the element that represents this Component.

```javascript
import {SimpleComponent} from "./simple-component";

/** A writable class holds a alias property to change the elements' textContent and extends SimpleComponent */
export class WritableComponent extends SimpleComponent {

    constructor(selector) {
        super(selector);
    }
    /** get proprety sets the function as a getter, you can use as writableComponent.textContent */
    get textContent() { return this.element.textContent; }

    /** The setter makes it that you can set it to a value, writableComponent.textContent = "some value" */
    set textContent(v) {
        return this.element.textContent = v;
    }
}
```

Finally, we'll create our ListComponent (`list-component.js`), which will have the ability to `getItem` from a pre-defined `items` property. Since a List shouldn't have its content via textContent, this Component extends SimpleComponent again.

```javascript
import {SimpleComponent} from "./simple-component";

/** A list component is a Component that its main functionality is to hold other components which need to be grabbed
 * in any form or situation. We do this by assigning each new component into a `items` property so that the
 * `getItem` statement can then retrieve it*/
export class ListComponent extends SimpleComponent {
    constructor(selector) {
        super(selector);
        this.items = [];
    }

    getItem(index) {
        if (typeof index !== "number") throw Error('getRow must have a number as an argument');
        if (index < 0 || index > this.items.length) throw Error('Out of Bounds');
        return this.items[index];
    }
}
```

## Game Components
Now that we have the base components set up, we can start formulating our visual components. The actual visual representation will be taken to another part of the code (the `game-hud.js` module). However, the actual creation, styling and functionality should be made at the component level, so our module only calls the methods available, instead of creating methods that no other module knows about.

Let's start by creating a `GameField` component which will be the visual representation of the actual game. We'll end by creating a separated component that will take care of our turn information.

Notice that We already know how the map is made, we already have a square generator function and we already have a ListComponent. Since we know that the field is nothing but a list of rows, with a list of columns, which compose "GameSlots" where we can write, we're in good shape for the project up ahead.

*Aww yeah! Take time to feel awesome. After all, thinking about design paid off. Maybe have a coffee and stretch out a bit.*

### GameSlot
We will create our GameSlot first - as that's the vital piece of starting things (as, it's the "clickable" square, per se). Create a new file, `game-slot.js` under `components` folder:
```javascript
import {WritableComponent} from "../libs/writable-component";

/** GameSlot is the class that's responsible to maintaign visual correlation between the GameEngine and the GameHud
 * Because this holds but the structure of the Component, we should refrain from making eventListeners here - unless
 * we want some special functionality that's not related to the HUD to happen (and even then, only if you can't write
 * that method into a Model or Service)
 * */
export class GameSlot extends WritableComponent {

    /** This class take slot as an argument; this will be made further on, on the GameEngine library
    * for now, we need only worry that this Object will have, in the future, to be populated with
    * {row: number, column: number} and that even further down, it will have a symbol property
    */
    constructor(slot) {
        super('game-slot');
        /** We set the style of the element after calling its super class and since we know this.element is a DOM Node
        * we can call all the browser functionalities on it - namely setAttibute()
        */
        this.element.setAttribute('style', "height: 60px; width: 60px; background-color: grey; display: inline-block; " +
            "border: 1px solid black; margin: 5px; font-size: large; color: black; line-height: 60px;" +
            "text-align: center; cursor: pointer");

        this.element.setAttribute('slot-row',slot.row);
        this.element.setAttribute('slot-column',slot.column);
        this.element.textContent = "-";
    }

    /** Again, overwriting the base class method because we want it to do more than just setting the value  */
    set textContent(slot) {
        super.textContent = slot.symbol;
        this.element.style.backgroundColor = 'white';
    }
}
```
### GameRow
Now that we have our Writable Square, we need to put it inside a list of rows. For that we will create the `game-row.js`
Class, which is a ListComponent holding N total GameSlots:

```javascript
import {ListComponent} from "../libs/list-component";
import {GameSlot} from "./game-slot";

/** GameRow is responsible for iterating the rows and creating the respective GameSlot, much like GameField */
export class GameRow extends ListComponent {

    /** Again, we will speak about these arguments further on, for now: row is an Array of slots
     * which can be iterated
     */
    constructor(row) {
        super('game-row');
        let gameSlot;
        this.items = [];
        this.element.setAttribute('style', "display: block;");

        row.forEach(slot => {
            gameSlot = new GameSlot(slot);
            this.element.appendChild(gameSlot.element);
            this.items.push(gameSlot);
        });
    }
}
```

### GameField
Almost done with our field, lets create `game-field.js` which is another ListComponent:

```javascript
import {GameRow} from "./game-row";
import {ListComponent} from "../libs/list-component";

/** GameField is responsible for creating the actual elements of the game.
 * We do this by iterating over the provided field from GameEngine and create a row and its columns with GameRow and
 * GameSlot */
export class GameField extends ListComponent  {
    constructor(field) {
        super('game-field');
        let gameRow;
        this.items = [];
        this.element.setAttribute('style', "font-family: Monospace; text-align: center");

        field.forEach((row) => {
            gameRow = new GameRow(row);
            this.element.appendChild(gameRow.element);
            this.items.push(gameRow);
        });
    }
}
```
## Turn Information and User Interaction
While our GameField is complete, we still need the visual cues that tell us whose turn it is, if the game has started, and if the game is over. 

Let's create our `Notice` class, which is a Simple Component that creates a background-pane, presents the user with a message, and deletes itself (and the pane) after a specific time interval.

Starting with `components/background-page.js`, which is a Simple Component with styling:
```javascript
import {SimpleComponent} from "../libs/simple-component";

/** A Simple semi-transparent back-pane */
export class BackgroundPane extends SimpleComponent {
    constructor() {
        super('background-pane');
        this.element.setAttribute('style', "position: absolute; top: 0; height: 100%; width: 100%;"
            + "background-color: black; opacity: .5; z-index: 10");
    }
}

```

Which will be then imported and initialized by `components/notice.js`:

```javascript
import {BackgroundPane} from "./background-pane";
import {SimpleComponent} from "../libs/simple-component";

/** Notice is the component we use when we want to show any special message to the user.
 * We first create the backgroundPane and attach it so we have a nice, clean, separated information.
 * The constructor takes a sentence (the text to show) and an interval that defaults to 1s so the notice auto-destroys
 * itself after X seconds. */

export class Notice extends SimpleComponent {
    constructor(text, interval = 1000) {
        super('notice');

        /** Before initializing a notice, check for the existance of a previous one and delete that so we dont
         * overposition text that the user wont be able to read */
        let element = document.querySelector(this.selector);
        if (element) this.removeElements();

        this.element.setAttribute('style', "position: absolute; top: 20%; background-color: white; z-index: 11;" +
            "text-align: center; font-family: Monospace; font-size: 25px; width: 100%;");
        this.element.textContent = text;
        this.backgroundPane = new BackgroundPane();

        document.body.appendChild(this.backgroundPane.element);
        document.body.appendChild(this.element);

        setTimeout(() => this.removeElements(), interval);
    }

    removeElements() {
        this.backgroundPane.destroy();
        this.destroy();
    }
}
```

### Handling turn information

With that out of the way, let's now create two Writable Component files that will be responsible for handling turns. We'll need one for Current Turn and another for PlaySymbol. Later, we will want a "wrapper component", so everything is tidy in its place.

Create `components/current-turn.js` and have it use `WritableComponent`:
```javascript
import {WritableComponent} from "../libs/writable-component";

/** This class holds the current turn number. Its overwriting its base class because we want to add more text than
 * the simple "value" that WritableComponent allows */
export class CurrentTurn extends WritableComponent {
    constructor() {
        super('current-turn');
        this.element.setAttribute('style', "float: left");
    }

    set textContent(v) {
        super.textContent = `turn No: ${v}`
    }
}
```

Create `play-symbol.js` inside `components/` folder,

```javascript
import {WritableComponent} from "../libs/writable-component";

/** Much like CurrentTurn, it shouws the Symbol that is in-play */
export class PlaySymbol extends WritableComponent {
    constructor() {
        super('symbol');
        this.element.setAttribute('style', 'float: right');
    }
    set textContent(v) {
        super.textContent = `playing: ${v}`
    }
}
```
Wrap it up with Turn Information (`turn-information.js`), another Simple Component, which will be controlled by the
GameHud module:

```javascript
import {SimpleComponent} from "../libs/simple-component";
import {PlaySymbol} from "./play-symbol";
import {CurrentTurn} from "./current-turn";

/** TurnInformation is the class that wraps CurrentTurn and Symbol into a whole component.
 * It's not a extension of ListComponent because we don't need to grab props by index and we can safely name them
 * to access them normally */
export class TurnInformation extends SimpleComponent {
    constructor() {
        super('turn-information');
        this.currentTurn = new CurrentTurn();
        this.symbol = new PlaySymbol();
        this.element.setAttribute('style', "text-transform: uppercase; font-size: 30px; height: 40px; display: block; font-family: Monospace");

        this.element.appendChild(this.currentTurn.element);
        this.element.appendChild(this.symbol.element);
    }

    /** An alias that updates both components */
    update(turn, symbol) {
        this.currentTurn.textContent = turn || 0;
        this.symbol.textContent = symbol || "";
    }
}
```

### Binding this all for the user to see
And to glue this elements all into a single view, we'll use the GameHud, `game-hud.js`, module; Create it in the `modules/` folder. This is a bigger file, but since we separated each functionality is just a matter of glueing things together.

You will see references to the `GameEngine` class. Quickly skip ahead and read about it in the next section. Then come back and follow along with the code below.

```javascript
import {GameEngine} from "../libs/game-engine";
import {GameField} from "../components/game-field";
import {Notice} from "../components/notice";
import {TurnInformation} from "../components/turn-information";

/** The GameHud class is what wraps all Componenets into the DOM as well as being responsible for the interaction
 * between the use interface and the Game Engine */
export class GameHud {
    constructor() {
        this.turns = null;
        this.gameEngine = null;
        this.gameField = null;
        this.turnInfo = new TurnInformation();

        document.body.appendChild(this.turnInfo.element);
        this.createGameField(false);

    }

    createGameField(lastWinner = 'x') {

        this.turns = 0;
        this.gameEngine = new GameEngine(['x', 'o'], lastWinner);

        /** if the GameField component exists in the page, remove that child from the body so we can
         * insert a new, clean, one. */
        const oldGameField = document.querySelector('game-field');
        if (oldGameField) document.body.removeChild(oldGameField);

        /** create a new GameField component */
        this.gameField = new GameField(this.gameEngine.field);

        /** We attach the click event to our GameSlots so we can click and have them actually work */
        this.gameField.items.forEach(row => {
            row.items.forEach(slot => {
                slot.element.addEventListener('click', (element) => this.occupyField(element))
            });
        });

        /** Append the GameField element to the body of the page */
        document.body.appendChild(this.gameField.element);

        /** Show a notice to the use and update the turn information via our TurnInfo alias */
        new Notice(`Game Start! First to Play: ${this.gameEngine.turnOf}`, 3000);
        this.turnInfo.update(this.turns, this.gameEngine.turnOf);
    }

    get isGameEnd() {
        return this.gameEngine.isWinner || this.gameEngine.isTie;
    }

    /** The HUD Processing of game end is responsible for showing the game end notices and restart the game after Xseconds */
    processGameEnd() {
        let winner = false;
        if (this.gameEngine.isWinner) {
            new Notice(`Game End! Winner is ${this.gameEngine.turnOf}! Game took ${this.turns} turns`, 1500);
            winner = this.gameEngine.turnOf;
        } else if (this.gameEngine.isTie) {
            new Notice(`Game End! It's a Tie! Game took ${this.turns} turns`, 1500);
        }

        setTimeout(() => {
            this.createGameField(winner);
        }, 1500);
    }


    /** Occupying a field in the HUD is made by calling the same function on the GameEngine end of things and working
     * with the output to update the interface. We grab the slot row/column from the clicked element and then
     * send it to GameEngine.
     *
     * We know that if occupying a field returns false then the occupation is impossible, we show that to the user.
     * Otherwise, we increment the turn, check if it's game end and if true process it, otherwise call the turn toggle
     * function from gameEngine and update the TurnInformation element. */
    occupyField(element) {
        let coords = {
            row: parseInt(element.target.getAttribute('slot-row'),10),
            column: parseInt(element.target.getAttribute('slot-column'),10)
        };

        let turnAction = this.gameEngine.occupyField(coords);
        if (!turnAction) {
            new Notice('This field is already occupied');
            return;
        }

        this.turns++;

        if (this.isGameEnd) {
            this.processGameEnd();
        } else {
            this.gameEngine.toggleTurn();
            this.turnInfo.update(this.turns, this.gameEngine.turnOf);
        }

        this.gameField.getItem(turnAction.row).getItem(turnAction.column).textContent = turnAction;
    }
}
```

## GameEngine & Win Conditions
The GameEngine library is akin to actually playing the game on paper. It tracks whose turn it is, which symbol to play on which square, if a square is playable, etc. It also has two aliases to the WinConditions Class. That way, we can separate the usage of WinCondition solely to the GameEngine, as it's the GameEngine that's supposed to know if the game is over (win or tie).

### Building the Game's Engine

#### WinConditions

Hop onto `libs/` and create the `WinConditions` class (`win-conditions.js`) first, as we already know how the map is made:
```javascript
import {createSquare} from './map';
/** Now that we have our field and our "customized slots", we can define "win conditions" pretty easily */

export class WinCondition {
    constructor(field = createSquare(3)) {
        this.field = field;
    }

    /** First, lets define the easiest ones: horizontals. We should iterate over each row and check if all of the columns
     * are occupied with the same symbol. If every column on one row is of the provided symbol, it will return true and
     * we then return the inspectingRow with all the information necessary to the HUD
     */
    horizontalLine(symbol) {
        let inspectingRow = [];
        return this.field.some(row => {
            inspectingRow = row;
            return row.every(slot => slot.occupied && slot.symbol == symbol);
        }) && inspectingRow;
    }

    /** The vertical ones, while a bit more tricky are easy enough as well: This is made by checking that for any given row
     * in the field it has the length of one of the columns occupied and of symbol. We achieve this by creating a temporary
     * array that aggregates every Nth column of each row and then checking if every element inside passes a condition.
     */

    verticalLine(symbol) {
        return this.field.some((row, index) => {
            let inspectingColumn = [];
            for (let x = this.field.length - 1; x > -1; x--) {
                inspectingColumn.push(this.field[x][index]);
            }
            return inspectingColumn.every(slot => slot.occupied && slot.symbol == symbol) && inspectingColumn;
        });
    }

    /** Now I guess you can imagine how we will clear the diagonals, just apply the same technique that we used on
     * verticalLines and increment index on each iteration as well and we should have it! nope :D
     *
     * This would make you check for combinations that wouldn't satisfy a "win" term as there would be groups of two and
     * groups of three slots. we are only interested in the latter. Furthermore, we need to check for diagonals that come
     * from right to left.
     *
     * So: we know a couple of things from diagonals: they can't take up more space than that that's available on a area
     * (or, the sum of two catheters equals the hypotenuse - guess those math classes were for something after all. All Hail
     * Pythagoras!)
     *
     * This is translated to: if the sum (or subtraction) of the row you are at and the length of the field is not equal to
     * the length of the field (or 0, in case of subtracting) that diagonal is not a win conditional.
     *
     * Now, we can simplify this even further because *we know* that diagonals can only become win conditions IF they are
     * from each corner (as those are the only places where a "full diagonal" condition exist) and we know that the middle
     * of the field *has* to be crossed by either line, so we can safely assume that if that slot isn't of the symbol, then
     * the symbol has no win condition on diagonals.
     *
     */

    diagonalLine(symbol) {
        const length = this.field.length - 1;
        const middle = length / 2;

        /** We first check if the middle, and one of the corners, are unoccupied if that's true then we can assume no win condition
         * is available at the time and return false */
        if (!this.field[middle][middle].occupied && (!this.field[length][0].occupied || !this.field[0][0].occupied)) return false;

        /** we now check which column is occupied and which symbol it contains so we can traverse its diagonal 
            This is a number value. */
        let column = (this.field[0][0].occupied && this.field[0][0].symbol === symbol) ? 0
            : (this.field[0][length].occupied && this.field[0][length].symbol === symbol) ? length : false;

        /** since we assigned a number to our column, is we have any other value (Not a Number) then we can assume neither
         * of the corners is occupied, returning false again for an early escape */
        if (typeof column !== "number") return false;


        /** Otherwise, lets traverse its diagonal, first checking which diagonal it is (last or fist) so we can increment or
         * decrement the for condition which is responsible to push the slots to the temporary array */
        let inspectingDiagonal = [];
        let row = 0;
        if (column === 0) {
            for (column; column <= length; column++) {
                inspectingDiagonal.push(this.field[row][column]);
                row++;
            }
        } else {
            for (column; column >= 0; column--) {
                inspectingDiagonal.push(this.field[row][column]);
                row++
            }
        }

        /** finally, if every item inside the temporary array pass the test, we have ourselves a diagonal win :) */
        return inspectingDiagonal.every(slot => slot.occupied && slot.symbol == symbol) && inspectingDiagonal;
    }

    /** The only case missing is that of a tie. When does a tie exist? Whenever all slots are occupied but no winner is
     * declared. We can then assume that: if there's one that's not occupied - it's not a tie.
     * since we do not care to know which elements are occupied with what, we can flatten the field array and traverse
     * it with the `every` array statement. */
    get tieExists() {
        const flatten = arr => arr.reduce((a, b) => a.concat(Array.isArray(b) ? flatten(b) : b), []);
        const flattenedField = flatten(this.field);
        return flattenedField.every(slot => slot.occupied === true);
    }

    /** create an alias for easy usage */
    hasLine(symbol) {
        return this.horizontalLine(symbol) || this.verticalLine(symbol) || this.diagonalLine(symbol);
    }

}
```
#### GameEngine class

Let's join this functionality onto the `game-engine.js` (in `libs/`):

```javascript
/**
 * Created by Mosh Mage on 1/22/2017.
 */
"use strict";
import {WinCondition} from './win-conditions';
import {createSquare} from './map';

/** With the map and Win Conditions out of the way, what's left for us to do is the Game Engine itself.
 * This class should take care of which turn is it, field occupation and if provided symbol won and what line was made */
export class GameEngine {
    constructor(symbols, lastWinner) {
        if (!symbols || symbols.length !== 2) throw Error('A game must be made of two symbols');
        this.turnOf = (lastWinner) ? lastWinner : (Math.round(Math.random()) === 0) ? symbols[0] : symbols[1];
        this.symbols = symbols;
        this.field = createSquare(3);
        this.winCondition = new WinCondition(this.field);
    }

    /** We created a square of 3 by 3, so we know anything beyond that is out of bounds; As well as anything bellow zero
     * so lets just go ahead and create a return statement that translate into that.
     * but first, lets check for the validity of the arguments - lets we throw an error further down because we allowed some
     * illegal input.
     * */
    isOutOfBounds(coords) {
        if (!coords || typeof coords.row !== "number" || typeof coords.column !== "number") return true;
        return coords.row > 3 || coords.row < 0 || coords.column > 3 || coords.column < 0;
    }

    isTurnOf(symbol) {
        return this.turnOf === symbol;
    }

    toggleTurn() {
        return this.turnOf = (this.turnOf === this.symbols[0]) ? this.symbols[1] : this.symbols[0];
    }

    /**
     * update the slot state to that of an occupied one with the symbol matching the turn order.
     * @param coords {Object} {row: number, column: number}
     * @returns {Object || Boolean} {occupied: boolean, symbol: string, row: number, column: number}
     */
    occupyField(coords) {
        if (this.isOutOfBounds(coords)) return false;

        let slot = this.field[coords.row][coords.column];
        if (slot.occupied) return false;

        slot.occupied = true;
        slot.symbol = this.turnOf;
        return slot;
    }

    /** A simple alias to call winCondition.hasLine with the provided symbol */
    get isWinner() { return this.winCondition.hasLine(this.turnOf) }
    get isTie() { return this.winCondition.tieExists; }

}
```
### Wrapping the GameStart
We wrap all the code we made in  the `game-start.js` file, located at the root folder (`src/`), by importing the
GameHud class and initializing it when we receive the DOM Ready event from the DOM.


```javascript
import {GameHud} from './modules/game-hud';
document.addEventListener("DOMContentLoaded", () => new GameHud());
 ```

All that's left to do now is a reference to the bundled file in our only html file, `index.html` (under `src/` folder):
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TicTacToe</title>
</head>
<body>
<script src="bundle.js"></script>
</body>
</html>
```
# Test your result!
Now, all we are left to do is play a few rounds!

Hop onto your Terminal, change directory into the project folder, bundle your code, and then open `dist/index.html` in
your browser.

```logtalk
$> cd tic-tac-toe
$> npm run bundle
```
---

That's it! We have made a full ES6 tic-tac-toe game.

In the next article, I'll cover:
- refactoring Simple Component to deal with Styling
- re-styling our game with a css pre-processor
- setting up a game-server and enable online-play
- deploying on heroku

Please leave your comments in the comments section, or feel free to email me at moshmage@gmail.com. Thank you for reading!