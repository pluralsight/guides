Bu makale [Globalization with Attached Properties in WPF](http://tutorials.pluralsight.com/microsoft-net/globalization-with-attached-properties-in-wpf) isimli makalemin Türkçe çevirisidir.

Doktora tezimin bir parçası olarak Windows üzerinde çalışan bir uygulamaya ihtiyaç duydum ve bir Windows Presentation Foundation (WPF) uygulaması yarattım. Ancak uygulamayı bitirdikten sonra bir problem ile yüzyüze geldim.

Tezimin dili İngilizce, ancak uygulama Türkiye'de kullanılacağından Türkçe olmalıydı. Bu sebeple uygulamaya çoklu dil desteği eklemeye karar verdim.

Çoklu dil desteğini sağlamak için bir çok yol mevcut. Örnek olarak; ```string``` tipinde kaynaklar oluşturmak, çeviriyi barındıran statik sınıflar oluşturmak yada çoklu dil seçeneği sağlayan kütüphaneler kullanmak. Ancak ben daha kolay ve Extensible Application Markup Language (XAML) ile uyumlu bir çözüm istiyordum. Mesela, şöyle basit bir kod satırı ile problemi çözmek istiyordum:

```xaml
<TextBlock Text=”English Text” English=”English Text” Turkish=”Türkçe Yazı” />
```

Biraz araştırmadan sonra "attached property" adında XAML'da özellik tanımlamaya yarayan bir konsept ile karşılaştım.

### Attached Property Nedir?

Bir [MSND makalesine](https://msdn.microsoft.com/en-us/library/ms749011%28v=vs.100%29.aspx) göre; "Attached property, Extensible Application Markup Language (XAML) tarafından tanımlanmış, bir konsepttir. Herhangi bir obje üzerine belirlenebilen attached property'nin, global tipte bir özellik olarak kullanılması amaçlanmıştır. Windows Presentation Foundation'da (WPF), geleneksel property wrapper'ı olmayan eklenmiş özellikler tipik olarak dependency property'lerin özelleştirilmiş bir formu olarak tanımlanmıştır."

Yani, attached property aslında WPF'de her gün kullandığımız property'ler ile aynıdır. Örnek olarak:

```xaml
<DockPanel>
   <CheckBox DockPanel.Dock="Top">Hello</CheckBox>
</DockPanel>
```

Örneğe bakacak olursak Dockpanel.Dock property'si aslında bizim için daha önceden tanımlı bir attached property'dir.

Attached property'leri kullanarak daha önce tanımlı property'ler gibi kendiniz istediğiniz özelliklerde bir property tanımlayabilirsiniz.

### Çoklu Dil Desteğinde Attached Property'ler

Attached property'leri bulduktan sonraki yeni hedefim, WPF kontrollerinin text, header ve/veya tooltip property'lerini otomatik olarak kullanıcının ayarladığı dile çevirecek bir attached property yaratmak oldu.

Bir örnek olarak, bu yazıda, attached property'leri ```TextBlock``` kontrolü ile kullanmayı göstereceğim.

Attached property'ler aslında extension method'lar şeklinde çalışır. Bu sebeple, çoklu dil desteği için, öncelikle aşağıdaki kodda olduğu gibi bir extension method tanımlamamız gerekir:

```cs
namespace SomeNamespace.Extensions
{
	public class UiExtensions
	{
		public static readonly DependencyProperty TurkishTextProperty = DependencyProperty.RegisterAttached("TurkishText",
                                                                                                        typeof(string),
                                                                                                        typeof(UiExtensions),
                                                                                                        new PropertyMetadata(default(string)));
        public static readonly DependencyProperty EnglishTextProperty = DependencyProperty.RegisterAttached("EnglishText",
                                                                                                        typeof(string),
                                                                                                        typeof(UiExtensions),
                                                                                                        new PropertyMetadata(default(string)));																							
																										
        public static void SetTurkishText(UIElement element, string value)
        {
            element.SetValue(TurkishTextProperty, value);
        }

        public static string GetTurkishText(UIElement element)
        {
            return (string) element.GetValue(TurkishTextProperty);
        }

        public static void SetEnglishText(UIElement element, string value)
        {
            element.SetValue(EnglishTextProperty, value);
        }

        public static string GetEnglishText(UIElement element)
        {
            return (string) element.GetValue(EnglishTextProperty);
        }
	}
}
```
Bu kodda; öncelikle ```TurkiskText``` ve ```EnglishText``` şeklinde ve ```string``` tipinde İngilizce ve Türkçe için 2 property tanımlıyoruz. Daha sonra bu property'ler için getter ve setter metotlarını tanımlıyoruz.

Yeni property'lerimizi tanımladığımıza göre, bunları XAML kodumuza ekleyebiliriz:

```xaml
<Window
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
		xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
		xmlns:uiExtensions="clr-namespace:SomeNamespace.Extensions"
		mc:Ignorable="d" x:Class="SomeNamespace.MainWindow"
        Title="MainWindow">
</Window>
```
Böylece aşağıdaki şekilde yeni property'leri kullanabiliriz:

```xaml
<TextBlock Text="Example English Text" 
			uiExtensions:UiExtensions.TurkishText="Örnek Türkçe Yazı" 
			uiExtensions:UiExtensions.EnglishText="Example English Text"/>
```

Ancak, uygulamayı çalıştırdığımızda yazının değişmediğini görüyoruz. Bunun sebebi dili nasıl değiştirmesi gerektiğini daha uygulamaya söylemedik.

Bunun için ```LanguageChanger``` sınıfını ve bir kaç yardımcı sınıfı yazmamız gerekecek.

```cs
namespace SomeNamespace.Globalization
{
    public static class AppLanguages
    {
        public static string English = "English";
        public static string Turkish = "Türkçe";
    }
}
```

```cs
namespace SomeNamespace.Utilities
{
    public static class UiUtilities
    {
        public static IEnumerable<T> GetChildren<T>(DependencyObject depObj) where T : DependencyObject
        {
            if (depObj != null)
            {
                for (int i = 0; i < VisualTreeHelper.GetChildrenCount(depObj); i++)
                {
                    DependencyObject child = VisualTreeHelper.GetChild(depObj, i);
                    if (child != null && child is T)
                    {
                        yield return (T)child;
                    }

                    foreach (T childOfChild in GetChildren<T>(child))
                    {
                        yield return childOfChild;
                    }
                }
            }
        }
    }
}
```

```cs
namespace SomeNamespace.Globalization
{
    public class LanguageChanger
    {
        private readonly UIElement _element;
        private readonly string _language;
        
        public LanguageChanger(UIElement element, string language)
        {
            _element = element;
            _language = language;
        }
        
        public void Change()
        {
            var textBlocks = UiUtilities.GetChildren<TextBlock>(_element);            

            if (_language == AppLanguages.English)
            {
                foreach (var textblock in textBlocks)
                {
                    textblock.Text = UiExtensions.GetEnglishText(textblock);                    
                }                
            }
            else if (_language == AppLanguages.Turkish)
            {
                foreach (var textblock in textBlocks)
                {                   
					textblock.Text = UiExtensions.GetTurkishText(textblock);                    
                }                
            }
        }
    }
}
```

```LanguageChanger``` kodu tüm TextBlock'ları bularak hangi dil seçeneği aktifse o text property'sini o dile göre ayarlayacak.

Ancak, bir problem var. Uygulamamız TÜM TextBlock'ları değiştirmeye çalışacak. Eğer ```EnglishText``` yada ```TurkishText``` property'lerini bulamazsa, boş string olarak değiştirecek ve biz bunu istemiyoruz. Bizim istediğimiz sadece çevrilmesi gereken TextBlock'ların çevrilmesi.

Bu problemi çözmek için ```LanguageChanger```sınıfında çeviri yapması ve yapmamasını işaretlemek için bir bayrak (flag) property'ye ihtiyacımız var.
 
Bunun için yeni bir attached property eklememiz gerekiyor:

```cs
namespace SomeNamespace.Extensions
{
	public class UiExtensions
	{
		public static readonly DependencyProperty TurkishTextProperty = DependencyProperty.RegisterAttached("TurkishText",
                                                                                                        typeof(string),
                                                                                                        typeof(UiExtensions),
                                                                                                        new PropertyMetadata(default(string)));
        public static readonly DependencyProperty EnglishTextProperty = DependencyProperty.RegisterAttached("EnglishText",
                                                                                                        typeof(string),
                                                                                                        typeof(UiExtensions),
                                                                                                        new PropertyMetadata(default(string)));
		public static readonly DependencyProperty ChangeLanguageProperty = DependencyProperty.RegisterAttached("ChangeLanguage",
																												typeof(bool),
																												typeof(UiExtensions),
																												new PropertyMetadata(default(bool)));																										
																										
        public static void SetTurkishText(UIElement element, string value)
        {
            element.SetValue(TurkishTextProperty, value);
        }

        public static string GetTurkishText(UIElement element)
        {
            return (string) element.GetValue(TurkishTextProperty);
        }

        public static void SetEnglishText(UIElement element, string value)
        {
            element.SetValue(EnglishTextProperty, value);
        }

        public static string GetEnglishText(UIElement element)
        {
            return (string) element.GetValue(EnglishTextProperty);
        }	
		
        public static void SetChangeLanguage(UIElement element, bool value)
        {
            element.SetValue(ChangeLanguageProperty, value);
        }

        public static bool GetChangeLanguage(UIElement element)
        {
            return (bool) element.GetValue(ChangeLanguageProperty);
        }
	}
}
```

Bu property ```bool``` tipinde olacak ve ```LanguageChanger```sınıfına bir TextBlock'u çevirip çevirmemesini belirtecek.

Ayrıca ```TextBlock``` ile ilgili XAML kodunu da değiştirmemiz gerekecek:

```xaml
<TextBlock Text="Example English Text"
			uiExtensions:UiExtensions.ChangeLanguage="True" 
			uiExtensions:UiExtensions.TurkishText="Örnek Türkçe Yazı" 
			uiExtensions:UiExtensions.EnglishText="Example English Text"/>
```

```ChangeLanguage``` ```true```olduğunda TextBlock'u çevirecek, ```false``` olduğunda çevirmeyecek.

Son olarak ```LanguageChanger``` sınıfımızı da değiştirmemiz gerekiyor:

```cs
namespace SomeNamespace.Globalization
{
    public class LanguageChanger
    {
        private readonly UIElement _element;
        private readonly string _language;
        
        public LanguageChanger(UIElement element, string language)
        {
            _element = element;
            _language = language;
        }
        
        public void Change()
        {
            var textBlocks = UiUtilities.GetChildren<TextBlock>(_element);
            

            if (_language == AppLanguages.English)
            {
                foreach (var textblock in textBlocks)
                {
                    if (UiExtensions.GetChangeLanguage(textblock))
                    {
                        textblock.Text = UiExtensions.GetEnglishText(textblock);
                    }
                }                
            }
            else if (_language == AppLanguages.Turkish)
            {
                foreach (var textblock in textBlocks)
                {
                    if (UiExtensions.GetChangeLanguage(textblock))
                    {
                        textblock.Text = UiExtensions.GetTurkishText(textblock);
                    }
                }                
            }
        }
    }
}
```

Böylece yazdığımız sistem sayesinde sadece XAML üzerinden tüm TextBlock'larda çeviri yapabilmek mümkün oldu.

Sadece TextBlock'ları mı çevirebiliriz? Tabi ki hayır.

Attached property'leri tüm ```string``` tipindeki property'leri çevirmek için kullanabiliriz. Örnek olarak, header, context menu, content vb. Yukarıda yazdığımız koda uygun propety'leri ekleyip, daha sonra ```LanguageChanger```sınıfında bu kontrolleri bularak çevirmesini söyleyebiliriz. Daha sonra bu kontrollerin çevirilerini de XAML üzerinden kontrol edebiliriz.

### Sonuç
Attached property'ler WPF'in çok kullanışlı bir parçası olup, yeni property'ler yaratmada veya daha önceden tanımlanmış property'lere yeni özellikler katmada kullanılabilir. Ben kendi projemde attached property'lerin çoklu dil desteğinde çok faydalı olduğunu öğrendim. Umarım bu teknik sizlere de kendi projelerinizde yardımcı olur.


