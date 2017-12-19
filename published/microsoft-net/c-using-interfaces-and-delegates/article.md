Have you encountered code with a lot of if-else logic? Most of the time when we see that kind of logic, it's a strategy problem. Someone is trying to solve a problem using the most primitive language tool, the 'if' statement. 

However, when it comes to Object Oriented Programming, it's important to understand that the code needs to be reusable, testable, and scalable. That's where a Strategy Pattern comes in handy. Let's take a look at an example. (Examples are in C# but the concept can be applied to any Object-Oriented Language)

<br>
First, we will see the code for something that is written with simple 'if' statements.


```CSharp
public int Calculate(int x, int y)
{
	if(action == "Add")
    {
    	return x+y;
    }
    else if(action == "Sub")
    {
    	return x-y;
    }
    else if
      ......
      .... and so on.
}
```

However, there are problems with this approach:

1. If you were to add another Strategy (let's say Divide), you will have to change the code in the class, thereby modifying the class itself. However, that is not a good practice. It violates the principle of OCP (Open Closed Principle). OCP says that the classes are Open for Extension but NOT for Modification. 

2. Too many 'if' blocks down the line. 

3. Hard-coded strings. I am sure that nobody wants them. As a counter, we could use Enums. However, this would add another dependency, when a faster way exists.

4. Removing a strategy is time-consuming. 

Now let's use the Strategy Pattern - a pattern that lets you choose the strategy on the fly. The strategy depends on the type of the object. The principle behind this pattern is **"Program to an Interface and not to an Implementation."** That makes the resulting solution easy to test, extensible, and maintainable. 

We start by defining the Interface. 
```CSharp
public interface ICalculate
{
	 int Calculate(int x, int y);
}
  ```

Now we can implement this interface. 

```CSharp
public class Add : ICalculate
{
    #region ICalculate Members

    public int Calculate(int x, int y)
    {
        return x + y;
    }

    #endregion
}

public class Subtract : ICalculate
{
    #region ICalculate Members

    public int Calculate(int x, int y)
    {
        return x - y;
    }

    #endregion
}
```

This is how we call the code. 
```CSharp
ICalculate calculate = new Add(); int sum = calculate.Calculate(a , b);
```

Since the object is of type `Add`, we don't have to check for the type (as we did above) to call the appropriate function. So, at run time, the `Calculate` method from the class `Add` gets called. 

How about adding a new Strategy. It's as simple as adding another class. We just need to make sure that it implements the interface. Let's create a new strategy called `Multiply` that implements `ICalculate`. 

```CSharp
public class Multiply : ICalculate
{
    #region ICalculate Members

    public int Calculate(int num1, int num2)
    {
        return num1 * num2;
    }

    #endregion
}
```

Some observations:

1. This example does not violate the OCP. In fact, it upholds it because we do not modify a particular class; our strategy is a new class with a specifically defined function.

2. If we need to remove a strategy, we can remove that class from our code, and it won't affect any other code (like it would have in the legacy code - we would have had to remove that if-else statement). 

3. It has no hard-coded strings. 

### Bonus: Coding Strategy Pattern using delegates?

Coding Strategy Pattern using delegates is very simple.

```CSharp
public class Calculator
{
    public static int Add(int x, int y)
    {
        return x + y;
    }

    public static int Subtract(int x, int y)
    {
        return x - y;
    }

    public static int Multiply(int x, int y)
    {
        return x * y;
    }

    public static int Divide(int x, int y)
    {
        return x / y;
    }
}
 ```
 
<br>
Now, we can use Delegates to call this code. 
```CSharp
Func calculate = Calculator.Add;
int sum = calculate(100, 50);
```

or 
```CSharp
calculate = Calculator.Subtract;
int difference = calculate(100, 50);
 ```

Is this too many lines of code? Well, in that case, we can just code the method inline. **Now, we don't need the class Calculator.**
```CSharp
Func calculate;
calculate = (x, s) => x + s;
```

The primary difference between Delegates and Interfaces is that while delegates reduce the code base and increase readability of code, you have to be careful on how you use them otherwise you might end up sacrificing testability. Coding to interfaces is usually more reliable, even if it requires more code.
						