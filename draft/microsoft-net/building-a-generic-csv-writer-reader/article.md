### What is a CSV file and why we use it
A (**C**omma **S**eperated **V**alue) CSV file is a plain text file which holds data in table format. Each line is like a row in a table, and columns are seperated by a comma, hence the name comma sepeated value.

In some cases the file may not be comma seperated. Especially if you are using a 3rd party program(e.g. Microsoft Excell), depending on the culture of your machine "seperator" might be a different character such as **";"**. This is because of the decimal sepeator is different in different cultures. In some cultures decimal seperator is **"."** so the CSV seperator can be **","**. But in some cultures decimal seperator is **","** so the CSV file has to use **";"** as seperator.

Below is an example of such a file containing table of first names and last names of people:

```csv
murat,aykanat
john,smith
```

CSV format is very useful because it is a text file and any operating system can read it. So for example if your application creates the CSV file on a Windows machine, you can open it up and use it in a Linux machine. So it is a very portable and easily readable file format.

### Formatting of a CSV file
### Why write a custom CSV writer/reader instead of using a 3rd party tool?
### CSV Writer
#### Writing from an IEnumerable
#### Writing from a XML file
### CSV Reader
### Conclusion


