(**C**omma **S**eperated **V**alue) CSV file format is a very common way of storing large datasets in a simple and portable format. Also since it is plain text, it is very easy to make such a file programmatically in an application. There are libraries around for such tasks but if you want to customize or create something specific it is better to write your own implementation for full control.

In this guide I will explain how to write a generic CSV Writer/Reader that will automatically pick data from the public properties of the input objects and generate a CSV file.

Before we dive into the code let me explain what is CSV file format so we can better understand what's going on in the code.

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

For example if your locale is set to a european culture, such as fr-FR, default decimal seperator becomes "," and you need to use ";" in CSV file as column seperator:
```csv
3,5;2,5;5,4
4,5;6,7;8,9
```
However in a machine which has en-US set as default, since decimal seperator is "." by default, same CSV file would look like this:
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
This is useful when the columns are not differentiable. For example if columns are all numbers, and you send the file to a person who does not know what the columns mean. That would be very confusing for the other person.So it would be better if we include headers in those scenarios to indicate what those column of values mean.

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

### CSV Writer
### CSV Reader
### Conclusion