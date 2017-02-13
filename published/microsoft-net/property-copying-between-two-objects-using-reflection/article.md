## The problem
Sometimes you have an object which has a lot of public properties, but you only need a handful of these properties for a particular task, such as creating a CSV file with that object. For example, check out the classes below:

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

However if we have, say, 30 properties on the "parent" object and 15 properties on the "child" object, we have to write 15 lines of repetitious, boring, and clunky code. There must be another way!

## The Solution

There are two ways we can circumvent this dilemma:
- We can copy similarly named properties from the "parent" object to the "child" object using reflection.
- We can use attributes in the "child" object to "mark" them for the parent object to copy its values to "child" object using reflection.

#### Property Copying
Let's start with the simple one. We will create a class named `PropertyCopier` with static method `Copy` to copy public properties from the parent object to the child object. To make our task easier, we'll only copy similarly-named properties. 

In this method we will have two `foreach` loops iterating over child and parent object's public properties, constantly checking to see if their names and types match. Whenever we find a match, we will copy the value to the child object's corresponding property.

```csharp
public class PropertyCopier<TParent, TChild> where TParent : class
                                            where TChild : class 
{
    public static void Copy(TParent parent, TChild child)
    {
        var parentProperties = parent.GetType().GetProperties();
        var childProperties = child.GetType().GetProperties();

        foreach (var parentProperty in parentProperties)
        {
            foreach (var childProperty in childProperties)
            {
                if (parentProperty.Name == childProperty.Name && parentProperty.PropertyType == childProperty.PropertyType)
                {
                    childProperty.SetValue(child, parentProperty.GetValue(parent));
                    break;
                }
            }
        }
    }
}
```

We can see the usage below:
```csharp
var user = new User()
{
    Username = "murat",
    Address = "Some address string here",
    Name = "murat",
    Lastname = "aykanat"
};

var person = new Person();

PropertyCopier<User, Person>.Copy(user, person);

Console.WriteLine("Person:");
Console.WriteLine(person.Name);
Console.WriteLine(person.Lastname);
```
Output will be:
```
Person:
murat
aykanat
```

#### Property Matching

It is all good when we have similar names and types, but what if we want something more dynamic? We have often classes where copying is useful but names differ from class to class. Furthermore, our current copier gives us little control over which properties actually get copied over to the child class. We may want to avoid sending over certain properties, even if they match in name and type.

Here's an example of some of our current version's limitations:
```csharp
public class PersonMatch
{
    public string NameMatch { get; set; }
    public string LastnameMatch { get; set; }
}
```
The types match, but the names don't. Our original `PropertyCopier` would *not* work in this scenario.

#### The better way

We need a way to "mark" the properties that we want to be copied from the parent object. The best way to do it would be using attributes to mark these properties.

Attributes can be defined as metadata. They can be assigned to classes, properties, methods, and so on. They provide metadata on the object they are set on. For example, we can define the maximum length of a string property. When we set a string to the property, it will first check against the metadata we provided beforehand and it will either set it to the new value or not depending on the length of the new string value.

In our case, we will use attributes to define the parent property name. We will tell the code that this subclass property will get its value from the property in the parent instance.

First of all we need to create a class to define our attribute. It will be a simple class that derives from `Attribute` class. It will have a single public field that defines the matching property name in the parent object. Here is the code for our attribute:
```csharp
[AttributeUsage(AttributeTargets.Property)]
public class MatchParentAttribute : Attribute
{
    public readonly string ParentPropertyName;
    public MatchParentAttribute(string parentPropertyName)
    {
        ParentPropertyName = parentPropertyName;
    }
}
```
With our attribute defined, we can add the necessary attributes to our class that we had before:
```csharp
public class PersonMatch
{
    [MatchParent("Name")]
    public string NameMatch { get; set; }
    [MatchParent("Lastname")]
    public string LastnameMatch { get; set; }
}
```
All we have to do now check the attribute value of each property in the child object and find the matching name in the parent object:
```csharp
public class PropertyMatcher<TParent, TChild> where TParent : class 
                                                  where TChild : class
{
    public static void GenerateMatchedObject(TParent parent, TChild child)
    {
        var childProperties = child.GetType().GetProperties();
        foreach (var childProperty in childProperties)
        {
            var attributesForProperty = childProperty.GetCustomAttributes(typeof(MatchParentAttribute), true);
            var isOfTypeMatchParentAttribute = false;

            MatchParentAttribute currentAttribute = null;

            foreach (var attribute in attributesForProperty)
            {
                if (attribute.GetType() == typeof(MatchParentAttribute))
                {
                    isOfTypeMatchParentAttribute = true;
                    currentAttribute = (MatchParentAttribute) attribute;
                    break;
                }
            }

            if (isOfTypeMatchParentAttribute)
            {
                var parentProperties = parent.GetType().GetProperties();
                object parentPropertyValue = null;
                foreach (var parentProperty in parentProperties)
                {
                    if (parentProperty.Name == currentAttribute.ParentPropertyName)
                    {
                        if (parentProperty.PropertyType== childProperty.PropertyType)
                        {
                            parentPropertyValue = parentProperty.GetValue(parent);
                        }
                    }
                }
                childProperty.SetValue(child, parentPropertyValue);
            }
        }
    }
}
```
Let's try our code and see what we get:
```csharp
var user = new User()
{
    Username = "murat",
    Address = "Some address string here",
    Name = "murat",
    Lastname = "aykanat"
};

var personMatch = new PersonMatch();

PropertyMatcher<User, PersonMatch>.GenerateMatchedObject(user, personMatch);

Console.WriteLine("Person:");
Console.WriteLine(personMatch.NameMatch);
Console.WriteLine(personMatch.LastnameMatch);
```
Output will be:
```
Person:
murat
aykanat
```
#### Property Copying/Matching in extension methods
If we don't want to use `PropertyCopier` or `PropertyMatcher` classes we can define extension methods for type `object`. We will use the same code as above to fill the methods:
```csharp
public static class ObjectExtensionMethods
{
    public static void CopyPropertiesFrom(this object self, object parent) 
    {
        var fromProperties = parent.GetType().GetProperties();
        var toProperties = self.GetType().GetProperties();

        foreach (var fromProperty in fromProperties)
        {
            foreach (var toProperty in toProperties)
            {
                if (fromProperty.Name == toProperty.Name && fromProperty.PropertyType == toProperty.PropertyType)
                {
                    toProperty.SetValue(self, fromProperty.GetValue(parent));
                    break;
                }
            }
        }
    }

    public static void MatchPropertiesFrom(this object self, object parent)
    {
        var childProperties = self.GetType().GetProperties();
        foreach (var childProperty in childProperties)
        {
            var attributesForProperty = childProperty.GetCustomAttributes(typeof(MatchParentAttribute), true);
            var isOfTypeMatchParentAttribute = false;

            MatchParentAttribute currentAttribute = null;

            foreach (var attribute in attributesForProperty)
            {
                if (attribute.GetType() == typeof(MatchParentAttribute))
                {
                    isOfTypeMatchParentAttribute = true;
                    currentAttribute = (MatchParentAttribute)attribute;
                    break;
                }
            }

            if (isOfTypeMatchParentAttribute)
            {
                var parentProperties = parent.GetType().GetProperties();
                object parentPropertyValue = null;
                foreach (var parentProperty in parentProperties)
                {
                    if (parentProperty.Name == currentAttribute.ParentPropertyName)
                    {
                        if (parentProperty.PropertyType== childProperty.PropertyType)
                        {
                            parentPropertyValue = parentProperty.GetValue(parent);
                        }
                    }
                }
                childProperty.SetValue(self, parentPropertyValue);
            }
        }
    }
}
```

To demonstrate the extension methods, we can use the code below:
```csharp
var user = new User()
{
    Username = "murat",
    Address = "Some address string here",
    Name = "murat",
    Lastname = "aykanat"
};

var person = new Person();

person.CopyPropertiesFrom(user);

Console.WriteLine("Person:");
Console.WriteLine(person.Name);
Console.WriteLine(person.Lastname);
```
```csharp
var user = new User()
{
    Username = "murat",
    Address = "Some address string here",
    Name = "murat",
    Lastname = "aykanat"
};

var personMatch = new PersonMatch();

personMatch.MatchPropertiesFrom(user);

Console.WriteLine("Person:");
Console.WriteLine(personMatch.NameMatch);
Console.WriteLine(personMatch.LastnameMatch);
```
Outputs of both code snippets will be the same:
```
Person:
murat
aykanat
```
#### Conclusion
In this guide I explained two ways to copy properties from one object to another. If you face a similar situation in your projects, instead of writing multiple lines of code, you can just use these classes or extension methods to copy the needed properties from one object to the other object.

I hope this guide will be useful for your projects. Please feel free to post your ideas and feedback.

Happy coding!
