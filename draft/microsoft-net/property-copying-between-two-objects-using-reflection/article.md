### The Problem
There are times when you have an object which has a lot of public properties, but you only need a part of these properties for a task such as creating a CSV file with that object. To demonstrate this issue let's look at the following classes below:

```csharp
public class User
{
    public string Username{get;set;}
    public string Address{get;set;}
    public string Name{get;set;}
    public string Lastname{get;set;}
}

public class Person
{
    public string Name{get;set;}
    public string Lastname{get;set;}
}
```

For the purpose of simplicity I kept classes with a low number of properties, but I am sure you can imagine we can add a lot more properties to each class.

Now, for example, we already have a `User` object and we want to create a `Person` object for another task and copy `Name` and `Lastname` properties from the User object.

If we have a user object like this:

```csharp
var user = new User()
{
    Username = "murat",
    Address = "Some address string here",
    Name = "murat",
    Lastname = "aykanat"
};
```

What we can do:

```csharp
var person = new Person()
{
    Name = user.Name,
    Lastname = user.Lastname
};
```

However if we have 30+ properties on the "parent" object and 15+ properties on the "child" object, we have to write at least 15 lines of code which is not very elegant or convenient.

### The Solution

There are two ways we can achieve what we want:
- We can copy similarly named properties from the "parent" object to the "child" object using reflection.
- We can use attributes in the "child" object to "mark" them for the parent object to copy its values to it using reflection.

#### Property Copying
#### Property Matching
#### Property Copying/Matching in extension methods
#### Conclusion