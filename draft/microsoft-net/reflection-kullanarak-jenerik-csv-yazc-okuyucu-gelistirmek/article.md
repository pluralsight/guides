Bu makale [Building a Generic CSV Writer/Reader using Reflection](http://tutorials.pluralsight.com/microsoft-net/building-a-generic-csv-writer-reader-using-reflection) isimli makalemin Türkçe çevirisidir.

(**C**omma **S**eperated **V**alue (Virgülle Ayrılmış Değerler)) CSV dosya biçimi veri kümelerini basit ve taşınabilir bir biçimde kaydetmenin en sık kullanılan yoludur. Ayrıca, dosya düz metin olduğundan, tüm programlama dillerinde bu tip dosyalar oluşturmak oldukça kolaydır. Bu tip dosyaları okumak yada yazmak için bir çok kütüphane mevcuttur ancak özel bir şekilde CSV dosyası yazmak/okumak ve kütüphane üzerinde tam kontrol sağlamak için kendi uygulamanızı yazmanızda fayda vardır.

Bu makalede, yarattığımız objelerin public property'lerinden verileri otomatik olarak alabilecek ve bu verileri kullanabilecek bir CSV okuyucu/yazıcı nasıl geliştirebileceğinizi anlatacağım.

Kod yazmaya başlamadan önce, yazdığımız kodu daha iyi anlayabilmek için CSV dosya biçiminin ne olduğunu ve nasıl kullanıldığını anlatacağım.

### CSV dosyası nedir?
CSV dosyası verileri tablo biçiminde tutan bir düz metin dosyadır. Her satır, tablodaki bir satıra denk gelir. Sütunlar ise virgülle ayrılmıştır. Bu sebeple de adı virgülle ayrılmış değerlerdir.

Aşağıda bir grup kişinin sıra numaraları, isimleri ve soyadlarını tutan bir CSV dosyası örneği görüyoruz:

```csv
1,murat,aykanat
2,john,smith
```

CSV biçiminin en büyük faydası çünkü düz metin olduğundan dolayı tüm işletim sistemleri tarafından okunabilir olmasıdır. Örneğin, uygulamanız CSV dosyasını Windows işletim sistemi olan bir bilgisayarda yaratsa bile, bu dosyayı Linux işletim sistemi olan bir bilgisayarda açabilir ve kullanabilirsiniz. Bu sebeple CSV kolay okunabilir ve taşınabilir bir biçimdir.

### CSV dosyasının biçimi

#### Ayraç (Separator) sorunları
Bazı durumlarda dosya virgülle ayrılmış olmayabilir. Özellikle CSV dosyasını 3. parti bir yazılım (ör: Microsoft Excell) ile yarattıysanız, bilgisayarınızın işletim sistemi diline bağlı olarak "ayraç" karakteri `;` karakteri gibi farklı bir karakter olabilir. Bunun sebebi değişik kültürlerde ondalık ayraç karakterinin farklı olmasıdır. Bazı kültürlerde ondalık ayracı `.` 'dır ve o kültürde CSV ayracının `,` olmasında bir problem yoktur. Ancak ondalık ayracının `,` olduğu kültürlerde, CSV ayracı `;` gibi farklı bir karakter olmalıdır.

Örneğin, eğer bilgisayar yereliniz (locale) bir `fr-FR` yada `tr-TR` gibi bir avrupa kültürü ise, varsayılan ondalık ayracınız `,`'dır, bu sebeple CSV ayracınız `;` yada başka bir karakter olmalıdır.

```csv
3,5;2,5;5,4
4,5;6,7;8,9
```
Ancak eğer bilgisayarınız `en-US` ise, ondalık ayracı `.` olduğundan, varsayılan CSV aşağıdaki şekilde görünür:

```csv
3.5,2.5,5.4
4.5,6.7,8.9
```

#### Her bir satırdaki veri alanlarının sayısı
Veri alanlarının sayısındaki en kritik kural, her satırda eşit sayıda veri alanı olmasıdır. Eğer aynı sayıda veri alanı olmazsa CSV okuyucular yarattığınız dosyayı okuyamazlar.

Eğer bir veri alanını özellikle boş bırakmak istiyorsanız boş `string` değerini kullanabilirsiniz

```csv
1,,aykanat
2,john,smith
```

#### Veri alanındaki virgül
CSV dosyanızda metin verileri varsa, veri alanlarınızdan birinin içinde virgül olması kaç adet virgül varsa o kadar ekstra veri eklenmesi gibi bir soruna sebep olur ve CSV dosyasının okunmasında hata olması anlamına gelebilir.

```csv
1, Hello, world!
```
Yukarıdaki örnekte, birinci sütun 1 ve ikinci sütun `Hello, world!`'dür, ancak CSV okuyucu bu veriyi 3 sütuna bölerek `1`, `Hello`, ` world!` olarak okur.

Bu problemi çözmek için tırnak işareti kullanmamız gerekir.

```csv
1, "Hello, world!"
```

Böylece `Hello, world!` verisinin tek veri alanına olduğunu belirtmiş oluruz.

Tırnak işaretlerini virgül içermeyen `string` değerlerde de kullanabilirsiniz ancak bu gereksizdir.

```csv
1,"murat","aykanat"
1,"john","smith"
```

#### Veri alanında tırnak işareti

Veri alanlarımızda tırnak işareti kullanmamız gerektiğinde, çift tırnak işareti kullanarak, o tırnak işaretinin veriye dahil olduğunu belirtebiliriz.

```csv
1,murat,""aykanat""
2,john,""smith""
```
That would read as; name is `murat`, lastname is `"aykanat"`.

#### Başlıklar

Aşağıdaki şekilde sütunlara başlık ekleyebiliriz:

```csv
id,name,lastname
1,murat,aykanat
1,john,smith
```

Başlık eklemek eğer veri alanları ile ne demek istendiği belli değilse çok yararlı olabilir. Örneğin tüm veriler sayı ise ve bu dosyayı bu sayıların ne anlama geldiğini bilmeyen birine gönderiyorsak, başlıklar olmazsa bu kişinin hangi sayının ne anlama geldiğini anlaması çok zordur. Bu sebeple başlıkları kullanarak daha anlamlı veri kümeleri gönderebiliriz.

#### Daha fazla bilgi
Eğer CSV biçimi hakkında daha detaylı bilgi almak iserseniz, ilgili [Wikipedia makalesini](https://en.wikipedia.org/wiki/Comma-separated_values) inceleyebilirsiniz.

### Teori
Our idea is simple, we want to input an array of objects into our CSV Writer and output a CSV file. For the reading part, we want to input the file path and output an array of objects back into the memory.

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
- Unless explicitly defined by quotation marks left and right spaces must be trimmed, because in my experience unnecessary left and right spaces are usually a mistake by the person who input the values, especially if the values are copied from some other application. If we absolutely need left and right spaces, we can use quotation marks.
- Properties can be ignored with their index or their name.

### CSV Writer

According to our considerations above we need a method for each class, ```ToCsv()```, that we want to turn to CSV format. This can be done in 3 ways:
- Interface implementation ```ICsvable``` with a method ```ToCsv()```
- Override ```ToString()```
- Abstract base class ```CsvableBase``` having a virtual ```ToCsv()``` method

I didn't want to use the ```ToString()``` override because maybe we need it somewhere else in our code, and I want the ```ToCsv()``` to be a separate method so it will be clear what it is doing. Also, since the code will be same for each class, I will go with the abstract class way. However if your particular class is already inheriting from a certain base class, you must go with the interface implementation since you can't inherit from more than one base class in C#. You just need to copy and paste for each class you want to turn to CSV.

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
            output += properties[i].GetValue(this).ToString();
            if (i != properties.Length - 1)
            {
                output += ",";
            }
        }

        return output;
    }
}
```
So what we do here is simple:
- Get all public properties of this class as a collection of ```PropertyInfo```.
- Iterate over them, get their value and add it to the output and put a comma.
- If we reach the end, do not put a comma.

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

To pre-process commas, quotation marks, and special characters, let's add a pre-processing method to our base class.

```cs
public abstract class CsvableBase
{
    public virtual string ToCsv()
    {
        string output = "";

        var properties = GetType().GetProperties();

        for (var i = 0; i < properties.Length; i++)
        {
            output += PreProcess(properties[i].GetValue(this).ToString());
            if (i != properties.Length - 1)
            {
                output += ",";
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

We just need to modify our code a bit to achieve what we want. Let's add following methods to our ```CsvableBase``` class.

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
- Define a boolean ```isFirstPropertyWritten``` to see if we write the first property yet.
- Get all the properties as ```PropertyInfo``` using reflection.
- Iterate over the properties, checking if the current property name or index is in the ignore or filter list by ```isIgnore``` flag.
- If it is not, we check if the first property is written.
- If the first property is not written then, we add a comma to output, since we will keep adding properties after that.
- Preprocess and add the current property's value to output.
- Set ```isFirstPropertyWritten``` so that we will keep adding commas.

We can now test our code:

```cs
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,"murat","aykanat");
        Console.WriteLine("Ignore by property name:");
        Console.WriteLine("Ignoring Id property: " 
        + p.ToCsv(new []{"Id"}, true));
        Console.WriteLine("Ignoring Name property: " 
        + p.ToCsv(new[] { "Name" }, true));
        Console.WriteLine("Ignoring Lastname property: " 
        + p.ToCsv(new[] { "Lastname" },true));
        Console.WriteLine("Ignore by property index:");
        Console.WriteLine("Ignoring 0->Id and 2->Lastname: " 
        + p.ToCsv(new[] { 0,2 },true));
        Console.WriteLine("Ignoring everything but Id: " 
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
#### Properties that Derive from CsvableBase

So far we only used value types as our properties. However what happens if we have a reference type property which also is derived from ```CsvableBase```?
Below is an example of such scenario:
```cs
public class Address : CsvableBase
{
    public Address(string city, string country)
    {
        City = city;
        Country = country;
    }
    public string City { get; set; }
    public string Country { get; set; }
}
public class Person : CsvableBase
{
    public Person(int id, string name, string lastname, Address address)
    {
        Id = id;
        Name = name;
        Lastname = lastname;
        Address = address;
    }

    public int Id { get; set; }
    public string Name { get; set; }
    public string Lastname { get; set; }
    public Address Address { get; set; }
}
```
The idea is, while iterating over the properties in our code, we need to dedect if the type of the property derives from ```CsvableBase```. Then call the ```ToCsv()``` method of that object instance by using reflection.

To achieve this, we need to modify our ```ToCsv()``` methods:

```cs
public virtual string ToCsv()
{
    string output = "";

    var properties = GetType().GetProperties();

    for (var i = 0; i < properties.Length; i++)
    {
        if (properties[i].PropertyType.IsSubclassOf(typeof (CsvableBase)))
        {
            var m = properties[i].PropertyType
                    .GetMethod("ToCsv", new Type[0]);
            output += m.Invoke(properties[i].GetValue(this),
                                new object[0]);
        }
        else
        {
            output += PreProcess(properties[i]
                                .GetValue(this).ToString());
        }
        if (i != properties.Length - 1)
        {
            output += ",";
        }
    }

    return output;
}

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

                if (properties[i].PropertyType
                    .IsSubclassOf(typeof(CsvableBase)))
                {
                    var m = properties[i].PropertyType
                    .GetMethod("ToCsv", new Type[0]);
                    output += m.Invoke(properties[i].GetValue(this),
                                        new object[0]);
                }
                else
                {
                    output += PreProcess(properties[i]
                                .GetValue(this).ToString());
                }

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

                if (properties[i].PropertyType
                .IsSubclassOf(typeof(CsvableBase)))
                {
                    var m = properties[i].PropertyType
                            .GetMethod("ToCsv", new Type[0]);
                    output += m.Invoke(properties[i].GetValue(this),
                                        new object[0]);
                }
                else
                {
                    output += PreProcess(properties[i]
                                .GetValue(this).ToString());
                }

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

                if (properties[i].PropertyType
                    .IsSubclassOf(typeof(CsvableBase)))
                {
                    var m = properties[i].PropertyType
                            .GetMethod("ToCsv", new Type[0]);
                    output += m.Invoke(properties[i].GetValue(this),
                                        new object[0]);
                }
                else
                {
                    output += PreProcess(properties[i]
                                .GetValue(this).ToString());
                }

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

                if (properties[i].PropertyType
                    .IsSubclassOf(typeof(CsvableBase)))
                {
                    var m = properties[i].PropertyType
                            .GetMethod("ToCsv", new Type[0]);
                    output += m.Invoke(properties[i].GetValue(this),
                                        new object[0]);
                }
                else
                {
                    output += PreProcess(properties[i]
                                .GetValue(this).ToString());
                }

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
Now let's try our modified code:
```cs
class Program
{
    static void Main(string[] args)
    {
        var p = new Person(1,"murat","aykanat",
                            new Address("city1", "country1"));
        Console.WriteLine(p.ToCsv());
        Console.ReadLine();
    }
}
```

Output is displayed as:
```text
1,murat,aykanat,city1,country1
```

Property ignoring works the same way as before, however you need to include parameters through reflection. But since I want to keep things simple at this point, I will not add the ignore feature of properties that derive from ```CsvableBase```.

#### Generic Writer
Since we laid the groundwork in ```CsvableBase```, ```CsvWriter``` itself is very simple:

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
                new Person(1, "murat", "aykanat",
                            new Address("city1","country1")),
                new Person(2, "john", "smith", 
                            new Address("city2","country2"))
            };

            var cw = new CsvWriter<Person>();
            cw.WriteFromEnumerable(people, "example.csv");
	}
}
```
If we check our application folder we can see our newly created file.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7753b093-4390-499d-9d05-474212141036.png)

If you open the file you can see output as expected:
```text
1,murat,aykanat,city1,country1
2,john,smith,city2,country2
```

### CSV Reader
Now, we have to reverse the process to read from our CSV file. To do this, we will follow what we did while reading. By this logic, as we implemented ```ToCsv()```, we need another method to reverse the CSV process in the ```CsvableBase``` class. Let's call this method ```AssignValuesFromCsv()```. Maybe not the most creative name, but we will go with that for now.
In this method we will do what we did in ```ToCsv()``` method. First, we will check whether the current property is derived from CsvableBase or not. After that, we will save the data back to the public properties.

```cs
public virtual void AssignValuesFromCsv(string[] propertyValues)
{
    var properties = GetType().GetProperties();
    for (var i = 0; i < properties.Length; i++)
    {
        if (properties[i].PropertyType
            .IsSubclassOf(typeof (CsvableBase)))
        {
            var instance = Activator.CreateInstance(properties[i].PropertyType);
            var instanceProperties = instance.GetType().GetProperties();
            var propertyList = new List<string>();

            for (var j = 0; j < instanceProperties.Length; j++)
            {
                propertyList.Add(propertyValues[i+j]);
            }
            var m = instance.GetType().GetMethod("AssignValuesFromCsv",                               new Type[] { typeof(string[]) });
            m.Invoke(instance, new object[] {propertyList.ToArray()});
            properties[i].SetValue(this, instance);

            i += instanceProperties.Length;
        }
        else
        {
            var type = properties[i].PropertyType.Name;
            switch (type)
            {
                case "Int32":
                    properties[i].SetValue(this,
                                    int.Parse(propertyValues[i]));
                    break;
                default:
                    properties[i].SetValue(this, propertyValues[i]);
                    break;
            }
        }
    }
}
```
Here we:

- Get all public properties of the object.
- Iterate over the properties.
- Check if the current property is derived from ```CsvableBase```.
- If so, create a temporary instance of that object.
- Get its properties.
- Call ```AssignValuesFromCsv()``` with its properties.
- If the property is not derived from Csvable base, just assign it to the property value according to the ```switch```.

You may notice we don't have ```float```, ```double``` or ```char``` in your switch statement. That's because in this example we only have ```int``` and ```string``` so I didn't want to make the class bigger.

So, now we have to iterate over the object via our ```CsvReader``` class.
```cs
public class CsvReader<T> where T : CsvableBase, new()
{
    public IEnumerable<T> Read(string filePath, bool hasHeaders)
    {
        var objects = new List<T>();
        using (var sr = new StreamReader(filePath))
        {
            bool headersRead = false;
            string line;
            do
            {
                line = sr.ReadLine();
                
                if (line != null && headersRead)
                {
                    var obj = new T();
                    var propertyValues = line.Split(',');
                    obj.AssignValuesFromCsv(propertyValues);
                    objects.Add(obj);
                }
                if (!headersRead)
                {
                    headersRead = true;
                }
            } while ( line != null);
        }

        return objects;
    }
}
```

Remember when I wrote that we may need ```ToString()``` override somewhere? Well, now we need it to print ```Person``` and ```Address``` objects. Also, we need to add an empty constructor for CSV Reader to work.

```cs
public class Address : CsvableBase
{
    public Address()
    {
        
    }
    public Address(string city, string country)
    {
        City = city;
        Country = country;
    }
    public string City { get; set; }
    public string Country { get; set; }
    public override string ToString()
    {
        return " " +City + " / " + Country;
    }
}
public class Person : CsvableBase
{
    public Person()
    {
        
    }
    public Person(int id, string name, string lastname, Address address)
    {
        Id = id;
        Name = name;
        Lastname = lastname;
        Address = address;
    }

    public int Id { get; set; }
    public string Name { get; set; }
    public string Lastname { get; set; }
    public Address Address { get; set; }
    public override string ToString()
    {
        return Name + " " + Lastname + " " + Address;
    }
}
```

Let's try our code:

```cs
class Program
{
    static void Main(string[] args)
    {
        var people = new List<Person>
        {
            new Person(1, "murat", "aykanat", new Address("city1","country1")),
            new Person(2, "john", "smith", new Address("city2","country2"))
        };

        var cw = new CsvWriter<Person>();
        cw.WriteFromEnumerable(people, "example.csv", true);

        var cr = new CsvReader<Person>();
        var csvPeople = cr.Read("example.csv", true);
        foreach (var person in csvPeople)
        {
            Console.WriteLine(person);
        }

        Console.ReadLine();
    }
}
```

Output:
```text
murat aykanat  city1 / country1
john smith  city2 / country2
```
### Conclusion
In this guide, I explained how you would develop your very own CSV writer and reader class. We used features of reflection to extract properties from classes and process them as needed, so we can just plug any class we want into our reader and writer. One of the benefits of generating your own CSV processing class is that you can modify it as you need different features so you don't get stuck with 3rd party libraries.

I hope this guide will be useful for your projects. Please feel free to post ideas and feedback.

Happy coding!
