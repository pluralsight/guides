Bu makale [Building a Generic CSV Writer/Reader using Reflection](http://tutorials.pluralsight.com/microsoft-net/building-a-generic-csv-writer-reader-using-reflection) isimli makalemin Türkçe çevirisidir.

(**C**omma **S**eperated **V**alue (Virgülle Ayrılmış Değerler)) CSV dosya biçimi veri kümelerini basit ve taşınabilir bir biçimde kaydetmenin en sık kullanılan yoludur. Ayrıca, dosya düz metin olduğundan, tüm programlama dillerinde bu tip dosyalar oluşturmak oldukça kolaydır. Bu tip dosyaları okumak yada yazmak için bir çok kütüphane mevcuttur ancak özel bir şekilde CSV dosyası yazmak/okumak ve kütüphane üzerinde tam kontrol sağlamak için kendi uygulamanızı yazmanızda fayda vardır.

Bu makalede, yarattığımız objelerin public property'lerinden verileri otomatik olarak alabilecek ve bu verileri kullanabilecek bir CSV okuyucu/yazıcı nasıl geliştirebileceğinizi anlatacağım.

Kod yazmaya başlamadan önce, yazdığımız kodu daha iyi anlayabilmek için CSV dosya biçiminin ne olduğunu ve nasıl kullanıldığını anlatacağım.

### CSV dosyası nedir?
CSV dosyası verileri tablo biçiminde tutan bir düz metin dosyasıdır. Her satır, tablodaki bir satıra denk gelir. Sütunlar ise virgülle ayrılmıştır. Bu sebeple de adı virgülle ayrılmış değerlerdir.

Aşağıda bir grup kişinin sıra numaraları, isimleri ve soyadlarını tutan bir CSV dosyası örneği görüyoruz:

```csv
1,murat,aykanat
2,john,smith
```

CSV biçiminin en büyük faydası düz metin olduğundan dolayı tüm işletim sistemleri tarafından okunabilir olmasıdır. Örneğin, uygulamanız CSV dosyasını Windows işletim sistemi olan bir bilgisayarda yaratsa bile, bu dosyayı Linux işletim sistemi olan bir bilgisayarda açabilir ve kullanabilirsiniz. Bu sebeple CSV kolay okunabilir ve taşınabilir bir biçimdir.

### CSV Dosyasının Biçimi

#### Ayraç (Separator) Sorunları
Bazı durumlarda dosya virgülle ayrılmış olmayabilir. Özellikle CSV dosyasını 3. parti bir yazılım (ör: Microsoft Excell) ile yarattıysanız, bilgisayarınızın işletim sistemi diline bağlı olarak "ayraç" karakteri `;` karakteri gibi farklı bir karakter olabilir. Bunun sebebi değişik kültürlerde ondalık ayraç karakterinin farklı olmasıdır. Bazı kültürlerde ondalık ayracı `.` 'dır ve o kültürde CSV ayracının `,` olmasında bir problem yoktur. Ancak ondalık ayracının `,` olduğu kültürlerde, CSV ayracı `;` gibi farklı bir karakter olmalıdır.

Örneğin, bilgisayar yereliniz (locale) bir `fr-FR` yada `tr-TR` gibi bir avrupa kültürü ise, varsayılan ondalık ayracınız `,`'dır, bu sebeple CSV ayracınız `;` yada başka bir karakter olmalıdır.

```csv
3,5;2,5;5,4
4,5;6,7;8,9
```
Ancak eğer bilgisayarınız `en-US` ise, ondalık ayracı `.` olduğundan, varsayılan CSV aşağıdaki şekilde görünür:

```csv
3.5,2.5,5.4
4.5,6.7,8.9
```

#### Her Satırdaki Veri Alanlarının Sayısı
Veri alanlarının sayısındaki en kritik kural, her satırda eşit sayıda veri alanı olmasıdır. Eğer aynı sayıda veri alanı olmazsa CSV okuyucular yarattığınız dosyayı okuyamazlar.

Eğer bir veri alanını özellikle boş bırakmak istiyorsanız boş `string` değerini kullanabilirsiniz

```csv
1,,aykanat
2,john,smith
```

#### Veri Alanındaki Virgül
CSV dosyanızda metin veri alanlarınızdan birinin içinde virgül olması, virgül sayısı kadar ekstra veri eklenmesi gibi bir soruna sebep olur ve CSV dosyasının okunmasında hata olması anlamına gelebilir.

```csv
1, Hello, world!
```
Yukarıdaki örnekte, birinci sütun 1 ve ikinci sütun `Hello, world!`'dür, ancak CSV okuyucu bu veriyi 3 sütuna bölerek `1`, `Hello`, ` world!` olarak okur.

Bu problemi çözmek için tırnak işareti kullanmamız gerekir.

```csv
1, "Hello, world!"
```

Böylece `Hello, world!` verisinin tek veri alanına ait olduğunu belirtmiş oluruz.

Tırnak işaretlerini virgül içermeyen `string` değerlerde de kullanabilirsiniz ancak bu gereksizdir.

```csv
1,"murat","aykanat"
1,"john","smith"
```

#### Veri Alanında Tırnak İşareti

Veri alanlarımızda tırnak işareti kullanmamız gerektiğinde, çift tırnak işareti kullanarak, o tırnak işaretinin veriye dahil olduğunu belirtebiliriz.

```csv
1,murat,""aykanat""
2,john,""smith""
```
Böylece, isim `murat`, soyad `"aykanat"` şeklinde okunur.

#### Başlıklar

Aşağıdaki şekilde sütunlara başlık ekleyebiliriz:

```csv
id,name,lastname
1,murat,aykanat
1,john,smith
```

Başlık eklemek eğer veri alanları ile ne demek istendiği belli değilse çok yararlı olabilir. Örneğin tüm veriler sayı ise ve bu dosyayı bu sayıların ne anlama geldiğini bilmeyen birine gönderiyorsak, başlıklar olmazsa bu kişinin hangi sayının ne anlama geldiğini anlaması çok zordur. Bu sebeple başlıkları kullanarak daha anlamlı veri kümeleri gönderebiliriz.

#### Daha Fazla Bilgi
Eğer CSV biçimi hakkında daha detaylı bilgi almak iserseniz, ilgili [Wikipedia makalesini](https://en.wikipedia.org/wiki/Comma-separated_values) inceleyebilirsiniz.

### Teori
Yazma işleminde dosyaya yazdırmak istediğimiz objeleri CSV yazıcıya yüklemek, okuma işleminde ise dosyayı okuyup, okunan veri kümesini objelere dönüştürmek istiyoruz.

**Giriş**
```cs
public class Person
{
    public int Id{ get; set };
    public string Name { get; set; }
    public string Lastname { get; set; }
}
```
**Çıkış**
```csv
1,murat,aykanat
2,john,smith
```
Ancak, göz önünde bulundurmamız gereken bazı kurallar vardır. Bu kurallar:
- CSV yazıcı ve okuyucu CSV dosya biçimine uygun yazmalı ve okumalı.
- Yazma ve okuma işlemi otomatik olmalı. Örneğin, bir sınıfta 2 public property bulunuyorsa, bunları elle yazmak kolay olabilir. Eğer 100 yada 10000 adet public property bulunuyorsa bunları tek tek el ile dosyaya yazmak mümkün değildir.
- Her obje kendi public property'lerini bir metot ile çıktı olarak vermelidir.

İsteğe bağlı kurallar:
- Bazı CSV okuyucuları UTF-8 kodlamayı desteklemediği için eğer mümkün ise tüm özel karaklerler ASCII biçimine çevrilmelidir. Bazı dillerde bu mümkün değildir, ancak bu makalede Türkçe karakterleri kullanacağımız için ASCII'ye çevirebileceğiz.
- Tırnak işaretleriyle özellikle belirtilmemişse, tüm sağ ve sol boşluklar silinmelidir. Dosyanın düzeni açısından, bu boşlukların genelde bilgileri giren kişiler tarafından yanlışlıkla yapıldığından dolayı silinmesi önemlidir.
- İstenen property'ler sıra sayısı veya ismi kullanılarak yazılmadan geçilebilmelidir.

### CSV Yazıcı
Kurallarımıza göre, her CSV biçime dönüştürmek istediğimiz sınıf için `ToCsv()` şeklinde bir metoda ihtiyacımız var. Bu 3 şekilde yapılabilir:
- Arayüz (Interface) uygulaması 
- `ToString()` metodunu override etmek
- `CsvableBase` absract sınıfı yaratıp, bu sınıfa virtual `ToCsv()` metodu eklemek

`ToCsv()` metodunun adından ne yaptığı anlaşılmasından dolayı ve `ToString()`'in kodun başka bir yerinde de işe yarayabileceğini düşünerek `ToString()`'i burada kullanmadım. Ayrıca tüm CSV'ye çevirmek istediğimiz sınıflarda `ToCsv()` metodu olması gerektiği için abstact sınıf tekniğini kullanmak istedim. Ancak, eğer CSV'ye çevirmek istediğiniz sınıf veya sınıflar başka bir sınıftan inherit ediyorsa ve C#'da iki ayrı sınıftan türetemeyeceğimiz için arayüz tekniğini kullanabilirsiniz. Ancak bu durumda `ToCsv()`metodunu kopyala/yapıştır şeklinde gereken tüm sınıflara yazmanız gerekir.

#### Basit Uygulama
Burada yapmamız gereken reflection kullanarak sınıfın tüm public property'lerini bulmaktır.
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

Burada yaptığımız:
- Sınıfıtım tüm public property'lerini `PropertyInfo`olarak alıyoruz.
- Döngü ile değerlerini alıp, çıkış değişkenine ekleyip, son olarak da virgül karakterini ekliyoruz.
- Eğer sona ulaşmışsak, yukarıdaki adımı virgül koymadan gerçekleştiriyoruz.

Şimdi kodumuzu yarattığımız `Person` sınıfı üzerinde bir console uygulaması ile deneyelim.
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
Yukarıdaki kodun çıktısı:
```text
1,murat,aykanat
```
#### Virgül, Tırnak İşareti ve Özel Karakterler

Virgül, tırnak işareti ve özel karakterleri düzgün bir biçimde okuyabilmek ve yazabilmek için değerleri bir ön işlemden geçirmek gerekiyor. Şimdi ön işleme kodumuzu ekleyelim.

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
`Person` objesini yeni metodumuzu test etmek için değiştirelim.
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
Bu kodun çıktısı:
```text
1,"""Hello"", world!",IGUCOiusoc
```
#### Özellikleri Eklemek veya Çıkarmak

Bazen, bir sınıftaki tüm public property'lere ihtiyaç duyulmaz. Bu sebeple ihtitaç duymadığımız property'leri sıra numarası veya adı ile reflection ile aldığımız property listesinden çıkarmalı yada ekleyebilmeliyiz.

Ancak başta yada sonda olması gibi kenar durumlarda (edge case) ne yapmalıyız? Virgülleri nasıl işleyeceğiz?

Biraz önce yazdığımız kodu değiştirerek bu problemleri çözebiliriz. Şimdi aşağıdaki metotları `CsvableBase` sınıfına ekleyelim.

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

Burada:
- ```isFirstPropertyWritten``` adında bir `bool` değişken ile ilk property'yi yazıp yazmadığımızı kontrol ediyoruz.
- Tüm property'leri reflection kullanarak alıyoruz.
- Döngü ile bu property'lerin isimlerinin veya sıra numaralarının ekleme yada silme listesinde olup olmadığını `isIgnore` değişkeni ile kontrol ediyoruz.
- Eğer değilse ilk property'nin yazılıp yazılmadığını kontrol ediyoruz.
- Eğer ilk property yazılmamışsa, çıkış değişkenine virgül ekliyoruz.
- O andaki property'nin değerini ön işlemden geçiriyoruz.
- ```isFirstPropertyWritten``` 'ı gelen diğer property'lerden sonra virgül eklenmesi için `true` olarak değiştiriyoruz.

Şimdi yeni yazdığımız kodu test edebiliriz:

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

Çıktı:

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
#### CsvableBase Sınıfından Türeyen Property'ler

Şimdiye kadar value type cinsinden property'ler kullandık. Ama ya kullandığımız property de `CsvableBase`'den türüyorsa ne yapacağız?
Aşağıdaki örnekte böyle bir senaryo mevcut:

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
Bu senaryoda, reflection ile aldığımız property'ler de döngü ile tek tek o property'nin `CsvableBase`'den türeyip türemediğini belirlememiz ve daha sonra bu objenin `ToCsv()` metodunu çağırmamız gerekir.

Bunu başarabilmek için `ToCsv()` metodunu değiştirmemiz gerekir:

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
Şimdi değiştirdiğimiz kodumuzu deneyelim:
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

Çıktı:
```text
1,murat,aykanat,city1,country1
```

`CsvableBase`'den türeyen property'lerde ekleme ve çıkarma da otomatik olarak aynı şekilde çalışacaktır, ancak bu sefer `ToCsv()` metodunu çağırdığımız şekilde ekleme veya çıkarma kodunu da reflection ile çağırmamız gerekir. Bu özelliği makaleyi karmaşıklaştırmamak adına burada yazmıyorum.

#### Jenerik Yazıcı
`CsvableBase`'de gerekli tüm kodları yazdığımız için `CsvWriter` sınıfını yazmak çok kolay olacak:

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
Kodumuzu ilk örneğimiz üzerinde deneyelim:
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
Eğer uygulama dosyamıza bakarsak, yeni yaratılan dosyayı görebiliriz.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7753b093-4390-499d-9d05-474212141036.png)

Dosya açıldığında yazılan çıktının beklenildiği şekilde olduğunu görebilirsiniz:
```text
1,murat,aykanat,city1,country1
2,john,smith,city2,country2
```

### CSV Okuyucu
CSV dosyamızdan okumak için yaptığımız işlemleri tersine çevirmemiz gerekiyor. Bunu yapabilmek için, yazarken ne yaptığımızı takip edelim. Önce `ToCsv()` metodunu yazdık, bu mantıkla `CsvableBase` sınıfına bu işlemi tersine çeviren bir metot yazmamız gerekir. Bu metoda `AssignValuesFromCsv()` diyelim. Bu metotta önce o anki property'nin `CsvableBase`'den türeyip türemediğine bakacağız. Daha sonra, verileri public property'ler içine yazacağız.


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

Burada:
- Objenin tüm public property'lerini alıyoruz.
- Property'lere döngü ile tek tek erişiyoruz.
- Property'nin `CsvableBase`'den türeyip türemediğine karar veriyoruz.
- Eğer türemişse, geçici bir örneğini(instance) oluşturuyoruz.
- Yaratılan örneğin tüm property'lerini alıyoruz.
- ```AssignValuesFromCsv()``` metodunu property'lerinde çağırıyoruz.
- Eğer `CsvableBase`'den türemişse, switch ile property değerine atıyoruz.


Farkettiyseniz, switch ifademizde `float`, `double`, yada `char` mevcut değil. Bunun sebebi, bu örneği kısa ve anlaşılır tutmak istemem ve örneğimizde sadece `int` ve `string` bulunmasıdır.

Şimdi, objeleri aşağıdaki şekilde `CsvReader` sınıfı ile okuyabiliriz.

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

`ToString()`'i kodun başka bir yerinde kullanabiliriz diye yazdığımı hatırlarsınız. Şimdi `Person` ve `Address` objelerini yazdırmak için bu metodu kullanacağız. Ayrıca `CsvReader`'ın çalışabilmesi için boş bir constructor'a ihtiyacımız var.


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

Yazdığımız kodu deneyelim:

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

Çıktı:
```text
murat aykanat  city1 / country1
john smith  city2 / country2
```
### Sonuç
Bu makalede, bir CSV yazıcı ve okuyucuyu nasıl geliştirebileceğinizi anlattım. Kütüphanemizi geliştirken, property'leri bulmak ve işlemek için reflection'ı kullandık. Kendi kütüphanenizi yaratmanızın avantajlarından biri de özelliklerini kendinizin belirlemesi ve 3. parti kütüphanelerin desteklemediği dosya tiplerini kodunuzu değiştirerek işleyebilmenizdir.

Umarım bu makalenim projelerinize yardımı dokunur. Lütfen fikir ve eleştirilerinizi yorum kısmında yazınız.
