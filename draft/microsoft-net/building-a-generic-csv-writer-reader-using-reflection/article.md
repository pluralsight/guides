(**C**omma **S**eperated **V**alue) CSV file format is a very common way of storing datasets in a simple and portable format. Also since it is plain text, it is very easy to make such a file programmatically in an application. There are libraries around for such tasks but if you want to customize or create something specific it is better to write your own implementation for full control.

In this guide I will explain how to write a generic CSV Writer/Reader that will automatically pick data from the public properties of the input objects and generate a CSV file.

Before we dive into the code let me explain what is CSV file format so we have a better understading of what we are dealing with.

### What is a CSV file?
A CSV file is a plain text file which holds data in table format. Each line is like a row in a table, and columns are seperated by a comma, hence the name comma sepeated value.

Below is an example of such a file containing table of first names and last names of people:

```csv
1,murat,aykanat
2,john,smith
```

CSV format is very useful because it is a text file and any operating system can read it.For example if your application creates the CSV file on a Windows machine, you can open it up and use it in a Linux machine. So it is a very portable and easily readable file format.

### Formatting of a CSV file

#### Seperator Issues
In some cases the file may not be comma seperated. Especially if you are using a 3rd party program(e.g. Microsoft Excell), depending on the culture of your machine "seperator" might be a different character such as **";"**. This is because of the decimal sepeator is different in different cultures. In some cultures decimal seperator is **"."** so the CSV seperator can be **","**. But in some cultures decimal seperator is **","** so the CSV file has to use **";"** as seperator.

For example if your locale is set to a european culture, such as "fr-FR", default decimal seperator becomes "," and you need to use ";" in CSV file as column seperator:
```csv
3,5;2,5;5,4
4,5;6,7;8,9
```
However in a machine which has "en-US" set as default, since decimal seperator is "." by default, same CSV file would look like this:
```csv
3.5,2.5,5.4
4.5,6.7,8.9
```

#### Number of data fields in each row
The most critical rule is every row must contain the same number of data fields otherwise it would be impossible to read by any CSV reader also it would not make sense.

If you have an empty data field you can just use an empty string.

```csv
1,,aykanat
2,john,smith
```

#### Comma in a data field
If you have text values in your CSV file, you might run into a problem where there is a comma inside one of your rows which would pose a problem because that field would be divided from that comma and you would end up with an extra column in that row.
```csv
1, Hello, world!
```
In the above example our first column is 1 and the second is "Hello, world!", however it a CSV reader would be divide the row into 3 columns 1, Hello and world!.

To solve this issue we must use quotation marks:
```csv
1, "Hello, world!"
```

This way we mean that the string Hello, world! is a single data field.

You can also use quotation mark on single word strings but it is not necessary.
```csv
1,"murat","aykanat"
1,"john","smith"
```

#### Quotation marks in a data field

We can also have actual quotation marks in our data fields. In this case we need to double our quotation marks indicating it is included in the data field.
```csv
1,murat,""aykanat""
2,john,""smith""
```
That would read as; name is murat, lastname is "aykanat".

#### Headers
You can also add headers to the columns.
```csv
id,name,lastname
1,murat,aykanat
1,john,smith
```
This is useful when the columns are not clear by its data fields. For example if columns are all numbers, and you send the file to a person who does not know what the columns mean, that would be very confusing for the other person because he or she doesn't know what those numbers mean.So it would be better if we include headers in those scenarios to indicate what those column of values mean.

#### More Details

If you would like to know more about CSV file format, you can use the [Wikipedia article](https://en.wikipedia.org/wiki/Comma-separated_values) about CSV and its resources.

### The Theory
Our idea is simple, we want to input an array of objects into our CSV Writer and output a CSV file. For the reading part we want to input the file path and output an array of objects back into the memory.

**Input**
```cs
public class Person
{
    public int Id{ get; set };
    public string Name { get; set; }
    public string Lastname { get; set; }
}
```
**Output**
```csv
1,murat,aykanat
2,john,smith
```
However there are some considerations:
- CSV Writer and Reader must abide by the rules of CSV file format mentioned above
- The process of writing must be automated. What I mean by this is, if we have 2 public properties in a class, it is fairly easy to write those properties to a file. But what if we have 100 or 10000 public properties? We just can't write them one by one.
- Every object should output its public properties via a method.

Optional considerations:
- Since some CSV readers out there do not support UTF-8 encoding, all "special" characters must be converted into ASCII counterparts if possible. Note that this may not be possible in some languages, however since I will be giving Turkish special characters as an example, it will be possible in this guide.
- Unless explicitly defined by quotation marks left and right spaces must be trimmed, because in my experience unnecessary left and right spaces are usually a mistake by the person who input the values, especially if the values are copied from some other application. If we absolutely need left and right spaces we can use quotation marks.
- Properties can be ignored with their index or their name.

### CSV Writer

According to our considerations above we need a method for each class, ```ToCsv()```, that we want to turn to CSV format. This can be done in 3 ways:
- Interface implementation ```ICsvable``` with a method ```ToCsv()```
- Override ```ToString()```
- Abstract base class ```CsvableBase``` having a virtual ```ToCsv()``` method

I didn't want to use the ```ToString()``` override because maybe we need it somewhere else in our code and I want the ```ToCsv()``` to be a seperate method so it will be clear what it is doing. Also, since the code will be same for each class, I will go with the abstract class way. However if your particular class is already inheriting from a certain base class you must go with the interface implementation since you can't inherit from more than one base class in C#. You just need to copy and paste for each class you want to turn to CSV.

#### Basic Implementation
What we need to do here is use reflection to get all the values of public properties of the class.
```cs
public abstract class CsvableBase
{
    public virtual string ToCsv()
    {
        string output = "";

        var properties = GetType().GetProperties();

        for (var i = 0; i < properties.Length; i++)
        {
            if (i == properties.Length - 1)
            {
                output += properties[i].GetValue(this).ToString();
            }
            else
            {
                output += properties[i].GetValue(this).ToString() + ",";
            }
        }

        return output;
    }
}
```
So what we do here is simple:
1. Get all public properties of this class as a collection of ```PropertyInfo```.
2. Iterate over them, get their value and add it to the output and put a comma.
3. If we reach the end, do not put a comma.

Let's test this on the ```Person``` class we created earlier in a console application.
```cs
public class Person : CsvableBase
{
    public Person(int id, string name, string lastname)
    {
        Id = id;
        Name = name;
        Lastname = lastname;
    }

    public int Id { get; set; }
    public string Name { get; set; }
    public string Lastname { get; set; }
}
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,"murat","aykanat");
        Console.WriteLine(p.ToCsv());
        Console.ReadLine();
    }
}
```
Output of above code:
```text
1,murat,aykanat
```
#### Comma, Quotation Marks and Special Characters

To pre-process commas, quotation marks and special characters, let's add a pre-processing method to our base class.

```cs
public abstract class CsvableBase
{
    public virtual string ToCsv()
    {
        string output = "";

        var properties = GetType().GetProperties();

        for (var i = 0; i < properties.Length; i++)
        {
            if (i == properties.Length - 1)
            {
                output += PreProcess(properties[i].GetValue(this).ToString());
            }
            else
            {
                output += PreProcess(properties[i].GetValue(this).ToString())
                + ",";
            }
        }

        return output;
    }
    private string PreProcess(string input)
    {
        input = input.Replace('ı', 'i')
            .Replace('ç', 'c')
            .Replace('ö', 'o')
            .Replace('ş', 's')
            .Replace('ü', 'u')
            .Replace('ğ', 'g')
            .Replace('İ', 'I')
            .Replace('Ç', 'C')
            .Replace('Ö', 'O')
            .Replace('Ş', 'S')
            .Replace('Ü', 'U')
            .Replace('Ğ', 'G')
            .Replace(""", """")
            .Trim();
        if (input.Contains(","))
        {
            input = """ + input + """;
        }
        return input;
    }
}
```
Let's change our ```Person``` object into something that can test our pre-processing method.
```cs
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,""Hello", world!","İĞÜÇÖıüşöç");
        Console.WriteLine(p.ToCsv());
        Console.ReadLine();
    }
}
```
Output of this code:
```text
1,"""Hello"", world!",IGUCOiusoc
```
#### Ignoring Properties

Sometimes you may not need all the public properties in a class. So we need to ignore or filter properties either by their name or their index in the list of properties we acquire by reflection.

But what happens in the edge cases such as the beginning or the end? How do we manage commas?

We just need to modify our code a bit to achive what we want. Let's add following methods to our ```CsvableBase``` class.

```cs
public virtual string ToCsv(string[] propertyNames, bool isIgnore)
{
    string output = "";
    bool isFirstPropertyWritten = false;
    

    var properties = GetType().GetProperties();

    for (var i = 0; i < properties.Length; i++)
    {
        if (isIgnore)
        {
            if (!propertyNames.Contains(properties[i].Name))
            {
                if (isFirstPropertyWritten)
                {
                    output += ",";
                }
                output += PreProcess(properties[i].GetValue(this).ToString());

                if (!isFirstPropertyWritten)
                {
                    isFirstPropertyWritten = true;
                }
            }
        }
        else
        {
            if (propertyNames.Contains(properties[i].Name))
            {
                if (isFirstPropertyWritten)
                {
                    output += ",";
                }
                output += PreProcess(properties[i].GetValue(this).ToString());

                if (!isFirstPropertyWritten)
                {
                    isFirstPropertyWritten = true;
                }
            }
        }
    }

    return output;
}

public virtual string ToCsv(int[] propertyIndexes, bool isIgnore)
{
    string output = "";

    bool isFirstPropertyWritten = false;

    var properties = GetType().GetProperties();

    for (var i = 0; i < properties.Length; i++)
    {
        if (isIgnore)
        {
            if (!propertyIndexes.Contains(i))
            {
                if (isFirstPropertyWritten)
                {
                    output += ",";
                }

                output += PreProcess(properties[i].GetValue(this).ToString());

                if (!isFirstPropertyWritten)
                {
                    isFirstPropertyWritten = true;
                }
            }
        }
        else
        {
            if (propertyIndexes.Contains(i))
            {
                if (isFirstPropertyWritten)
                {
                    output += ",";
                }

                output += PreProcess(properties[i].GetValue(this).ToString());

                if (!isFirstPropertyWritten)
                {
                    isFirstPropertyWritten = true;
                }
            }
        }
        
    }

    return output;
}
```
Here, we:
1. Define a boolean ```isFirstPropertyWritten``` to see if we write the first property yet.
2. Get all the properties as ```PropertyInfo``` using reflection.
3. Iterate over the properties, checking if the current property name or index is in the ignore or filter list by ```isIgnore``` flag.
4. If it is not, we check if the first property is written.
5. If the first property is not written then we add a comma to output, since we will keep adding properties after that.
6. Preprocess and add the current property's value to output.
7. Set ```isFirstPropertyWritten``` so that we will keep adding commas.

We can now test our code:

```cs
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,"murat","aykanat");
        Console.WriteLine("Ignore by property name:");
        Console.WriteLine("Ignoring Id property: 
" 
        + p.ToCsv(new []{"Id"}, true));
        Console.WriteLine("Ignoring Name property: 
" 
        + p.ToCsv(new[] { "Name" }, true));
        Console.WriteLine("Ignoring Lastname property: 
" 
        + p.ToCsv(new[] { "Lastname" },true));
        Console.WriteLine("Ignore by property index:");
        Console.WriteLine("Ignoring 0->Id and 2->Lastname: 
" 
        + p.ToCsv(new[] { 0,2 },true));
        Console.WriteLine("Ignoring everything but Id: 
" 
        + p.ToCsv(new[] { "Id" }, false));
        Console.ReadLine();
    }
}
```

Output is:

```text
Ignore by property name:
Ignoring Id property:
murat,aykanat
Ignoring Name property:
1,aykanat
Ignoring Lastname property:
1,murat
Ignore by property index:
Ignoring 0->Id and 2->Lastname:
murat
Ignoring everything but Id:
1
```

Finally we are done with ```CsvableBase``` class and ```ToCsv()``` method, now we can move on to the writer itself.
#### Generic Writer
Since we did the groundwork in ```CsvableBase```, ```CsvWriter``` itself is very simple:

```cs
public class CsvWriter<T> where T : CsvableBase
{
	public void Write(IEnumerable<T> objects, string destination)
	{
		var objs = objects as IList<T> ?? objects.ToList();
		if (objs.Any())
		{
			using (var sw = new StreamWriter(destination))
			{
				foreach (var obj in objs)
				{
					sw.WriteLine(obj.ToCsv());
				}
			}
		}
	}

	public void Write(IEnumerable<T> objects, string destination, 
	                    string[] propertyNames, bool isIgnore)
	{
		var objs = objects as IList<T> ?? objects.ToList();
		if (objs.Any())
		{
			using (var sw = new StreamWriter(destination))
			{
				foreach (var obj in objs)
				{
					sw.WriteLine(obj.ToCsv(propertyNames, isIgnore));
				}
			}
		}
	}

	public void Write(IEnumerable<T> objects, string destination, 
	        int[] propertyIndexes, bool isIgnore)
	{
		var objs = objects as IList<T> ?? objects.ToList();
		if (objs.Any())
		{
			using (var sw = new StreamWriter(destination))
			{
				foreach (var obj in objs)
				{
					sw.WriteLine(obj.ToCsv(propertyIndexes, isIgnore));
				}
			}
		}
	}
}
```
Now let's try our code with our initial example:
```cs
class Program
{
	static void Main(string[] args)
	{
		var people = new List<Person>
		{
			new Person(1, "murat", "aykanat"),
			new Person(2, "john", "smith")
		};

		var cw = new CsvWriter<Person>();
		cw.Write(people,"example.csv");
	}
}
```
If we check our application folder we can see our newly created file.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7753b093-4390-499d-9d05-474212141036.png)

If you open the file you can see output as expected:
```text
1,murat,aykanat
2,john,smith
```

### CSV Reader
### Conclusion