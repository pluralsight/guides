CSV file format is a very common way of storing large datasets in a simple and portable format. Also since it is plain text, it is very easy to make such a file programmatically in an application. There are libraries around for such tasks but if you want to customize it is better to write your own implementation for full control.

### What is a CSV file and why we use it
A (**C**omma **S**eperated **V**alue) CSV file is a plain text file which holds data in table format. Each line is like a row in a table, and columns are seperated by a comma, hence the name comma sepeated value.

Below is an example of such a file containing table of first names and last names of people:

```csv
murat,aykanat
john,smith
```

CSV format is very useful because it is a text file and any operating system can read it. So for example if your application creates the CSV file on a Windows machine, you can open it up and use it in a Linux machine. So it is a very portable and easily readable file format.

### Formatting of a CSV file

#### Seperator Issues
In some cases the file may not be comma seperated. Especially if you are using a 3rd party program(e.g. Microsoft Excell), depending on the culture of your machine "seperator" might be a different character such as **";"**. This is because of the decimal sepeator is different in different cultures. In some cultures decimal seperator is **"."** so the CSV seperator can be **","**. But in some cultures decimal seperator is **","** so the CSV file has to use **";"** as seperator.

For example if your application does not explicitly set en-US culture, and if you are running your application in a machine which uses "," as seperator, CSV file would look like this:
```csv
3,5;2,5;5,4
4,5;6,7;8,9
```
However in a machine which has en-US set as default same output would be:
```csv
3.5,2.5,5.4
4.5,6.7,8.9
```

#### Comma in a data field
If you have text values in your CSV file, you might run into a problem where there is a comma inside one of your rows which would pose a problem because that field would be divided from that comma and you would end up with an extra column in that row.
```csv
1, Hello, world!
```
In the above example our first column would be 1 and the second is "Hello, world!", however it would be divided from the "Hello".

To solve this issue we must use quotation marks:
```csv
1, "Hello, world!"
```

This way we mean that "Hello, world!" is a single column.

You can also use quotation mark on single word strings but it is not necessary.
```csv
"murat","aykanat"
"john","smith"
```

#### Quotation marks in a data field

We can also have actual quotation marks we need to include in our data. In this case we need to double our quotation marks indicating it is included in the data field.
```csv
murat,""aykanat""
john,""smith""
```
That would read as; name is murat, lastname is "aykanat".

#### Headers
You can also add headers to the columns.
```csv
name,lastname
murat,aykanat
john,smith
```
This is useful when the columns are not differentiable. For example if columns are all numbers, and you send the file to a person who does not know what the columns mean, that would be very confusing.So it would be better if we include headers in those scenarios.

### Why write a custom CSV writer/reader instead of using a 3rd party tool?

### CSV Writer
#### Writing from an IEnumerable
#### Writing from a XML file
### CSV Reader
### Conclusion