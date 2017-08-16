#### Conditional Logic is all about the IF word. 
In fact, it's practically impossible to program effectively without using IF. You can write simple programs like a calculator. But for anything more complicated, you need to get the hang of Conditional Logic.

As an example, take a very basic calculator program. It only has a the plus button to handle addition. Let's say we add a minus button. Now, you as the developer can't say beforehand which of the two buttons your users will click. Do they want to add or subtract? You need to be able to write code that does the following:

> **IF** the Plus button was clicked, add

> **IF** the Minus button was clicked, subtract

You can rearrange the two statements above.

> Was the Plus button clicked? Yes, or No?

> Was the Minus button clicked? Yes, or No?

This allows us to reduce our problem. The answer for each is either going to be Yes or No. Either the button is either clicked or it is not clicked.


## IF Statements


To test for YES or NO values, you can use an IF statement. In C# .NET, set up the IF conditional like this:
```
if ( )
{

}
```
Between the parentheses, type what you want to check for (Was the button clicked?). In languages like C# .NET and Java, it's [customary (but not strictly necessary)](https://stackoverflow.com/questions/359732/why-is-it-considered-a-bad-practice-to-omit-curly-braces) to add a pair of curly brackets that will contain your actual code. This code determines what really happens IF the answer to your condition is YES. Here's an example:

```
bool buttonClicked = true; // a boolean variable can be either true or false

if (buttonClicked == true) // this is the CONDITION or the yes-no question
{

MessageBox.Show(“The button was clicked”); // this gets performed if the condition is met

}
```

Notice the first line of code:

```
bool buttonClicked = true;
```

This is a variable type you may not have met before - bool. The bool is short for Boolean. You use a Boolean variable type when you want to check for true or false values. **A Boolean variable can only ever be true or false.** Our bool variable above is `buttonClicked`. Its value has been set to true.

The next few lines are our IF Statement:

```
if (buttonClicked == true)
{

MessageBox.Show(“The button was clicked”);

}
```

The double equals sign (`==`) is a common convention when using IF Statements, and its ubiquitous throughout the CS world. It compares the terms on either side and checks if they have the same value.
In general, `==` compares [memory references](https://en.wikipedia.org/wiki/Reference_(computer_science). 

The double equals sign is an example of a **Conditional Operator**. There are a few others that you'll meet shortly. But the whole of the line reads:

 > "IF buttonClicked has a value of true"

If you miss out one of the equals signs, you'd have this:

```
if (buttonClicked = true)
```

In this case, you're assigning a value of true to the variable buttonClicked. The code above is *not* checking if buttonClicked "has a value of" true. This generally causes errors because most languages do not allow assignment in a conditional statement. Mastering the difference between `==` and `=` can eliminate lots of problems!

Between the curly brackets of the IF statement, we have a simple MessageBox line. But this line will only get executed IF buttonClicked has a value of true.

Let's try it out. Start a new C# project for this (File > New Project). Add a button to your new form, and set the Text property to "IF Statement". Double click the button, and add the code from above. So your coding window will look like this:


![alt text](http://i.imgur.com/5nXedhe.png)

Run your program and click the button. You should see the message box. 

Now halt the program and change this line:

```
bool buttonClicked = true;
```

to this

```
bool buttonClicked = false;
```

So the only change is from true to false. Run your program again, and click the button. What happens? Nothing!

The reason that nothing happens is that our IF statement is checking for a value of true:

```
if (buttonClicked == true)
```

C# will only execute the code between the curly brackets IF buttonClicked has a value of true. Since you changed its value to false, it doesn't bother with the MessageBox in between the curly brackets, and moves on to any remaining code.

 

## Else

You can also say what should happen if the answer was false. All you need to do is make use of the else word. You do it like this:

```
if (buttonClicked == true)
{

}
else
{

}
```

This allows you to write your code for what should happen IF the conditional statement is false. Change your code to this:

```
if (buttonClicked = = true)
{

MessageBox.Show("buttonClicked has a value of true"); // if true, do this.

}
else
{

MessageBox.Show("buttonClicked has a value of false"); // if false, do this.

}
```

So the whole thing reads:

>"IF buttonClicked has a value of true, do the first thing. ELSE, do the second thing."

Run your program, and click the button. You should see the second MessageBox display. It should read "buttonClicked has a value of false." Halt the program and change the first line back to true. So this:

```
bool buttonClicked = true;
```

instead of this:

```
bool buttonClicked = false;
```

Run the program again, and click the button. This time, the first message box will display. It should read "buttonClicked has a value of true."

The whole point of using If...Else Statements, though, is to execute one piece of code instead of some other piece of code.

You can also extend the IF statement and add an **Else If** part to make your code more complex.

I hope that you found this beginner-level introduction to conditionals useful as you build more difficult projects. Maybe now you can try your hand at a calculator app that performs a range of functions, such as plus, minus, multiply, and divide.
_________
Thank you for reading. Please leave comments and feedback in the discussion section below.