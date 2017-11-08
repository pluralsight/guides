## Introduction and Background
I have been a C# programmer for more than 4 years and have seen it change from C# 5 to C# 6. C# got evolved but most of the programmers have not yet evovled to follow the best standard based practices for their projects to make them more readable. The basic idea of this post is to allow you to be able to write good projects in case of your open source C# projects and, and .NET framework applications that run in a team environment. In such cases, writing a good code can be a plus point for the developers because the code being written is going to be used, managed and updated by the rest of the developers who are going to follow the philosophy and coding practices of that team ground. In these cases the best approach is somehow to follow the guidelines of the coding teams, but my post will give you an overview of how likely you should consider adding the design and style to the C# programs in your application projects to make them better for readers! Note that, C# compiler doesn't care about the style that you put into the codes, only the performance will be reduced (or increased) by changing the style of the code. 

Without saying anything else, let's get deeper into programming of C# applications in a way that they look simpler, cleaner and easier for the readers and at the same time maintaining their performance and efficiency. A few things that I would like you to know prior to reading the rest are:

1. Improvements to C# that were made in its 6th version. 
2. LINQ in .NET framework.
3. Asynchronous programming and `Task` object in C#.
4. Unsafe programming in C#, which allows you to go into memory management stuff. 

I will be guiding you through these few of the ways in which you can make your C# programs much better than they were before you read this guide! I will also try to make sure that I do not go against (any of) your in-door rules for coding and practices for better code in the teams. 

##### Not talking about the performance
It should be noted that I am not talking mostly about changing the program performance, upgrading the efficiency of decreasing the time taken by the programs to run. It _can_ improve the performance in a matter of seconds by writing good C# code, but all of the tips do not guarantee that your codes will be performing better, too. Because that field is not of _how you wrote the code, but what code you wrote_. That said, the code will still anyways help you in many the code resistant to breaking on the runtime, unless that is expected. 

## Why write clean code at all?
You write the code, compiler compiles with no warnings and no errors, the code is fine. But what about if some else wanted to read that code out? What is someone had to upgrade the code later for you, or the company you had worked for. Have a look at the following code, 

``` csharp
public static void Main(string[] args) {
   int x = 0;
   x = Console.Read();
   
   Console.WriteLine(x * 1.5);
}
```
That program works well, no errors in the system and well the application also sort of works. But can you tell me what does the program do, in real life? There are many assumptions:

1. It just multiplies the value. 
2. It is increasing the value, like in the case of a bonus. 
3. It is the interest ratio of the total value of person's bank balanace.
4. Etc. Etc. Etc.

Which one is real? No one will know, in these cases, it is better to write good codes and remember to follow the basics of the programming. Have a look at the following code, 

``` csharp
public static void Main(string[] args) {
   int salary = 0;
   salary = Console.Read();
   
   Console.WriteLine(salary * 1.5);
}
```
Doesn't this make more sense that the prior one? We can easily say that this code is going to increase the value of the salary. Note how, just by improving the code we were able to allow the to make more sense than before. 

In this guide, I won't show you how to follow best principles. Instead, I will start by building on the top of what you already have and teach you how to make the best out of your C# programs. I will teach you, how to write good C# logic in your applications, for sake of many benefits that you can gain from the applications, by writing programs in that manner and structures. So, let's get started. 

### Count the tips down...
It won't take much of your time, but attention that you want to give me in order to learn these basics rules to make your C# programs better.

#### Object intialization
C# is an object-oriented programming language, and what good there would be to write a set of tips, if there is no section for objects themselves. This section will talk about a few things that you should consider knowing, before moving forward and writing the `new Object()` code in your applications. You must be aware of how C# classes are created, how things collaborate to bring up a small program in your system. For example, have a look at the following code:

``` cshaop
class Person {
    public int ID { get; set; }
    public string Name { get; set; }
    public DateTime DateOfBirth { get; set; }
    public bool Gender { get; set; }
}
```
You may want to create the programs that set the values by default, or coming from a model or any other database-oriented data source like this source code, that simplifies the way you enter the defaults in the object at the time the object is being created. 

```
var person = new Person { ID = 1, Name = "Afzaal Ahmad Zeeshan", DateOfBirth = new DateTime(1995, 08, 29), Gender = true };
```
Instead of writing the same code in the following way, 

``` csharp
var person = new Person();
person.ID = 1;
person.Name = "Afzaal Ahmad Zeeshan";
// So on.
```
There is no notable performance improvement to the code here, but the readability of the code can really be improved. If you are an _indentation_ guy, have a look here, 

``` csharp
var person = new Person 
             {
                ID = 1, 
                Name = "Afzaal Ahmad Zeeshan",
                DateOfBirth = new DateTime(1995, 08, 29),
                Gender = true
             };
```
Which also has the indentation if this adds a bit more clarification to the readability to your C# code. Although, these do tjhe same thing but the ones being suggested make the code more readable and concise. 

#### Null check up
You ever get annoyed by the `NullReferenceException`, when an object that missed initialization comes back for revenge? There are many benefits of having the null checkups in your programs, not just for better readability, but also in case of ensuring that the program does not terminate due to memory issues&mdash;such as when a variable does not exist in the memory. These can also add up to the security of your programs and the good UI and UX guidelines that your teams may be having on the list. Most of the times, a null exception gets raised due to this:

``` csharp
string name = null;

Console.WriteLine(name);
```
In most cases, compiler itself won't continue until you fix this, but if you somehow manage to trick the compiler into thinking that the variable got the value but on the runtime there wasn't any. Null reference exception will come up. To overcome this, you can just do the following:

``` csharp
string name = null; 

// Try to enter the value, from somewhere
if(name != null) {
   Console.WriteLine(name);
}
```
This safe checkup will ensure that the value was available, when this variable was called. Otherwise, it will move in other direction and away from the buggy path. In C# 6, however, there is another way of overcoming this error. Consider a scenario, where your database was setup, your table of data was set up, your person was found, but their employment details were not found. Can you get the company where he worked?

``` csharp
var company = DbHelper.PeopleTable.Find(x => x.id == id).FirstOrDefault().EmploymentHistory.CompanyName; // Error
```
If you did this, there will be an error, because we were only able to move up a few steps in the list of these values. Then we ended up hitting a null value and everything was lost. C# 6 comes up with a new way of overcoming these cases, by using a safe navigation operator after the values and fields which can be null; `?.`. Like this. 

``` csharp
var company = DbHelper?.PeopleTable?.Find(x => x.id == id)?.FirstOrDefault()?.EmploymentHistory?.CompanyName; // Works
```
What this code would do is that it would only check for next value if the previous was not null. If the previous value was null, it will return null and save the null as the value for the `company`. Instead of throwing the error, this can come handy where you leave the checkups to the framework itself. But, nonetheless, you still have to check at the end if the rest of the values are null or not. 

``` csharp
var company = DbHelper.PeopleTable?.Find(x => x.id == id)?.FirstOrDefault()?.EmploymentHistory?.CompanyName; 

if(company != null) {
   // Final process
}
```
But you get the point, instead of writing the code and checking everything to be null, you can do that simple checked up and perform the actions and logics that you want to in your programs. Otherwise, there would be a need of `try...catch` wrapper or multiple `if...else` blocks to control how your program navigates in the system. 

#### Asynchronous pattern of programming
If you are programming using C#  5, then you are already using the async/await keywords to bring responsiveness to your applications but if that is not the case then I would recommend using an asynchronous programming pattern to your application's source code. This can not only just bring the responsiveness to your programs, but will also increase the readability to your application. A few of the benefits for having asynchronous pattern in your source code are:

1. Code paths start to make much more sense. Programmers can understand where would the program go if there is a process that starts running in the background. 
2. Application hang problems will go away, most of the application freeze related problems come directly from the code when UI thread cannot update the UI, users consider the application to be handing and not responding, whereas that is not the case. Asynchronous approach can really help a lot in this. 
3. Windows Runtime based applications are entirely based on this approach. You will be (**and must be!**) using this approach in one of your Windows RUntime applications to overcome problems like hanging applications, or bad programming practices. 

Ever since threading, parallelization of the code execution has come into existence, asynchrony has become a vital part of the programs and applications and thus you should also consider using this. 

#### String building in C&#35;
Strings are a vital part of applications nowadays and building the strings can take a lot of time and sometimes can cause a performance lag too in the applications. There are many ways in which you can build the strings in C# programs. Following are a few of the ways, 

```
string str = ""; // Setting it to null would cause additional problems.

// Way 1
str = "Name: " + name + ", Age: " + age;

// Way 2
str = string.Format("Name: {0}, Age: {1}", name, age);

// Way 3
var builder = new StringBuilder();
builder.Append("Name: ");
builder.Append(name);
builder.Append(", Age: ");
builder.Append(age);
str = builder.ToString();
```

Note that strings in C# are immutable. Which means that if you try to update their values, they are recreated and previous handles are removed from the memory. That is why, the first one may look like if that is the best way to go, but that is not. The best way to go to is the third way which allows you to build up the strings without having to recreate the objects in the memory. However, C# 6 has introduced an entirely new way of building the strings in C# and that are much better than the ones that you can image of. The new _String interpolation_ operator `$` provides you with the facility of performing the string building in best possible way. The string interpolation goes like this, 

``` csharp
static void Main(string[] args)
{
   // Just arbitrary variables
   string name = "";
   int age = 0;
   
   // Our interest
   string str = $"Name: {name}, Age: {age}";
}
```
Just a single line of code, and the compiler will automatically convert this one to the one we already see, `string.Format()` version. For a proof, I am going to read the bytecode that this C# program has generated and that will show you how this would automatically change the syntax to actually read the formatting of the string.

``` csharp
IL_0000:  nop         
IL_0001:  ldstr       ""
IL_0006:  stloc.0     // name
IL_0007:  ldc.i4.0    
IL_0008:  stloc.1     // age
IL_0009:  ldstr       "Name: {0}, Age: {1}"
IL_000E:  ldloc.0     // name
IL_000F:  ldloc.1     // age
IL_0010:  box         System.Int32
IL_0015:  call        System.String.Format
IL_001A:  stloc.2     // str
IL_001B:  ret         
```
As can be seen, this shows how the syntax is changed back to the one we have already seen. See the **`IL_0009`** for more. Besides, this also brings a cleaner look to your programs when someone else is reading the programs, at the same time it brings about a bit of performance if the strings to be built are smaller. In case of larger strings, use `StringBuilder`.

#### Looping over a large collection of data
What good would an application do if there is no looping and iteration over a collection of data, and in these conditons there  are times when you would have to find a value, find a node, find a record or perform any other traversing over the collection. In such cases you really need to make sure you are writing the good code as this is the area where performance and readability both matter a lot and they are _correlated_. I have been doing this myself and with a bit of experience I was able to overcome the wrong way of writing the code for reading and traversing the data. That is exactly where LINQ experience should jump in and allow you to write the program that use the best of .NET framework to provide the best coding experience and the best experience to your users and customers. 

Previously, you might have done a few of these:

``` csharp
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   foreach (var person in people) {
      if(person.ID == id) {
         return person;
      }
   }
   
   // No person found.
   return null;
}

// Then do this
var person = FindPerson(123);
```
This can be an easy read for anyone trying to come up in your area. But however, the code be made simpler and cleaner using LINQ queries in C#. There are two ways in which you can do this. One is a bit SQL-like and other is by using `Where` function over the collection and passing the requirement that we have. Let us have a look at both of them. 

```csharp
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   return (from person in people
          where person.ID == id
          select person).ToList().FirstOrDefault();
}

// Then do this
var person = FindPerson(123);
```
This code has a bit of SQL-like look and would enhance the readability and the performance of your code, The function is similar, however, this one reads things a bit better and leaves everything &mdash; iteration oriented &mdash; to .NET framework itself. .NET framework would do its best to provide best performance to the applications. Now, let us have a look at another way of writing this query in the same C# code,
```csharp
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   return people.FirstOrDefault(x => x.ID == id);
}

// Then do this
var person = FindPerson(123);
```
Notice that, the first code returned `null` in case when there was no match found. These codes also do the same thing. The only thing that code had bad was that it had to perform iterations over the collection itself. That local `return person;` would allow the program to return the control but what would happen if the data was at the last location? The complexity of this algorithm of data search would still be O(n). 

#### Avoid unsafe context
C# also supports manual memory management, during the times when you have to pilot your plane yourself! The unsafe context in C# allows you to manipulate the memory, perform pointer arithmetic, read and write the data at memory locations which may not be provided access to and so on. However, .NET framework does many things to overcome the memory issues, latency and other stuff that goes around on disk. That also allows .NET framework to overcome the need to actually perform any memory management at all, .NET framework would do that for you. 

Now there are many benefits of using unsafe context, such as when you want to write wrappers around the native C++ libraries. Emgu CV is one of such examples where you would like to write some of the code that handles how the native code stuff is managed and to handle the errors in the memory in a way they would be simpler. In this context you can do, 

1. Pointer management and pointer arithmetic. You cannot perform anything on the addresses outside of this context, that is where .NET rules. 
2. Memory management, manipulate the objects right in the memory. 
3. Use C++ style of programming; which is what C# was designed to be doing. 

So, there are either no benefits of this or very less benefits and if you should consider this in your applications, consider it wisely. 

###### Everything said about `unsafe` was personal view
I also want to notify that everything that I have said and the pros and cons that I have shared are my personal views and I have not very much used `unsafe` context in my programs because there is no reason that I should consider using it in my applications. However, if your applications require native memory management then you can use this context. 

#### Use lambda expressions where possible
Lambdas come from arena of functional programming, and in C# it has been widely used in many ways ranging from inline functions all the way down to (C# 6's)'s getter only properties. I will show both of these usages in C# that will make the program not only look cooler, but instead also have a better performance factor; _for that I will show you the IL of that C# code_. I personally enjoy using lambdas in many areas, specially where i have to write inline functions in C#. Ever since getter-only properties could be written by using this concept, I have been using them and personally saying it is better than the previous ways of doing the same thing. 

##### 1. Using lambdas for inline functions
You must be well known of the delegates of C# programming. There are many fiedls where they are used such as in the cases of event handling in your applications. For event handling, you can write your current functions like the following ones:

``` csharp
// Without lamdbas
myBtn.Click += Btn_Click;

public void Btn_Click (object sender, EventArgs e) {
   // Code to handle the event
}

// With the help of lambdas
myBtn.Click += (sender, e) =>
               {
                  // Code to handle the event.
               }
```
Note that compiler will automatically map the objects to their types. This is convenient in many ways as it allows you to write the inline functions in C# that remain with the objects only, unless you want to use them anywhere else too. There is however one disadvantage to this method of handling the events, the disadvantage is that you cannot remove the event handler once it has been attached. In C# you can `-+` but since we don't have any reference to remove the events, we are only left to use the separate functions. **But**, if you don't have to remove the handlers, you should always consider using this way of event handling in your programs. 

##### 2. Using lambdas for getter-only properties
In C#, there is a concept of using properties instead of fields. Where you control how a value is set, and how a value is captured from the field. Consider them an alternative (or similar ones) to the getter and setters of Java programming language. Only difference is that you don't have to write them separately somewhere, they are written right infront of the field itself. C# program compilers would then make their own backing fields that are used to store the values. 

Basically, you would have to write the getter-only properties like this:

``` csharp
public string Name { get; }
```
Note that these properties are constant and you cannot change them once they are set. They were set in the constructors, or (as of C# 6), they are set right infront of it. Like this, 

``` csharp
public string Name { get; } = "Afzaal Ahmad Zeeshan";
```
However, since we already know that this is a constant field and you cannot modify this, then why not create a simple constant property? This is where things get a bit tricky, even a property has to be backed up by a field. In which case, this would do the trick for us:

``` csharp
public string Name => "Afzaal Ahmad Zeeshan";
```
This is equivalent of writing the following, 

``` csharp 
public string Name { get { return "Afzaal Ahmad Zeeshan"; } }
```
But the performance is much better since the getter-only field is converted to a constant field and then that field is used in the program anytime this property has to be called.

### Final words
This is a post where you can find a few of the tips that can help you in making the most of your C# applications using best (or at least better) ways to write your C# programs and in which, the performance can also be fine-tuned. If you have more, let me know I would love to hear them out. :-) 

The purpose of this guide was to allow you guys to understand and see a few of the ways in which your programs can be made better in reading and performing. C# compiler itself does the most to increase the quality of your code, efficiency of your code whereas sometimes that is left on the programmer's will to make the program work better. There are many other ways of improving the readability, sometimes such of these are left on the company's way of writing the programs because most teams require programmers to follow their own programming methods and ways. The post also considers those standards and provides the ways in which you can write good code and still follow those guidelines. 