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
- Since some CSV readers out there do not support UTF-8 encoding, all local characters must be converted into ASCII format. This may not be possible in some languages, however since I will be giving Turkish as an example, it will be possible in this guide.
- Unless explicitly defined by quotation marks left and right spaces must be trimmed.
- Properties can be ignored with their index or their name.

### CSV Writer

First of all we need to plan this in a way so that we can write it once and use it everywhere. According to our considerations above we need a method for each class , ```ToCsv()```, that we want to turn to CSV. This can be done 2 ways:
- Interface implementation
- Override ToString()
- Abstract base class

I didn't want to use the ```ToString()``` override because maybe we may need it somewhere else and I wantthe ```ToCsv()``` to be a seperate method so it will be clear what it is doing. Since the code will be same for each class, I will go with the abstract class way. However if your particular class is already inheriting from a certain base class you can go with the interface. You just need to copy paste for each class you want to turn to CSV.

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
1 - Get all public properties of this class as a collection of ```PropertyInfo```.
2 - Iterate over them, get their value and add it to the output and put a comma.
3- If we reach the end, do not put a comma.

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

This is the basic implementation, however it does not yet satisfy other considerations we set such as special characters, commas and quotation marks. For that, let's add a pre-processing method to our base class.

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
        input = input.Replace('ı', 'i').Replace('ç', 'c')
                      .Replace('ö', 'o').Replace('ş', 's')
                      .Replace('ü', 'u').Replace('ğ', 'g')
                      .Replace('İ', 'I').Replace('Ç', 'C')
                      .Replace('Ö', 'O').Replace('Ş', 'S')
                      .Replace('Ü', 'U').Replace('Ğ', 'G')
                      .Replace("\"", "\"\"")
                      .Trim();
        if (input.Contains(","))
        {
            input = """ + input + """;
        }
        return input;
    }
}
```
Lets change our ```Person``` object into something that can test our pre-processing method.
```cs
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,"\"Hello\", world!","İĞÜÇÖıüşöç");
        Console.WriteLine(p.ToCsv());
        Console.ReadLine();
    }
}
```
Output of this code:
```text
1,"""Hello"", world!",IGUCOiusoc
```

### CSV Reader
### Conclusion