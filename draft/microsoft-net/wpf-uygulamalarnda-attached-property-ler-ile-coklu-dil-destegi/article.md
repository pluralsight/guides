Doktora tezimin bir parçası olarak Windows üzerinde çalışan bir uygulamaya ihtiyaç duydum ve bir Windows Presentation Foundation (WPF) uygulaması yarattım. Ancak uygulamayı bitirdikten sonra bir problem ile yüzyüze geldim.

Tezimin dili İngilizce, ancak uygulama Türkiye'de kullanılacağından Türkçe olmalıydı. Bu sebeple uygulamaya çoklu dil desteği eklemeye karar verdim.

Çoklu dil desteğini sağlamak için bir çok yol mevcut. Örnek olarak; string tipinde kaynaklar oluşturmak, çeviriyi barındıran statik sınıflar oluşturmak yada çoklu dil seçeneği sağlayan kütüphaneler kullanmak. Ancak ben daha kolay ve Extensible Application Markup Language (XAML) ile uyumlu bir çözüm istiyordum. Mesela, şöyle basit bir kod satırı ile problemi çözmek istiyordum:

```xaml
<TextBlock Text=”English Text” English=”English Text” Turkish=”Türkçe Yazı” />
```

Biraz araştırmadan sonra "attached property" adında XAML'da özellik tanımlamaya yarayan bir konsept ile karşılaştım.

### Attached Property Nedir?

Bir [MSND makalesine](https://msdn.microsoft.com/en-us/library/ms749011%28v=vs.100%29.aspx) göre; "Attached property, Extensible Application Markup Language (XAML) tarafından tanımlanmış, bir konsepttir. Herhangi bir obje üzerine belirlenebilen attached property, global tipte bir özellik olarak kullanılması amaçlanmıştır. Windows Presentation Foundation'da (WPF), geleneksel property wrapper'ı olmayan eklenmiş özellikler tipik olarak dependency property'lerin özelleştirilmiş bir formu olarak tanımlanmıştır."

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
Bu kodda; öncelikle ```TurkiskText``` ve ```EnglishText``` şeklinde ve ```string``` tipinde İngilizce ve Türkçe için 2 property tanımlıyoruz. Daha sonra bu property'ler için getter ve setter methodlarını tanımlıyoruz.

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

Bu problemi çözmek için ```LanguageChanger```sınıfında bir flag property'ye ihtiyacımız var.
To solve this issue we need a flag for the ```LanguageChanger``` class to decide whether or not the application should translate. 

For this we need yet another attached property:

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

The property we added is of type ```bool``` and it will help ```LanguageChanger``` whether to skip this ```TextBlock``` or not.

We also need to change our XAML code accordingly and add our newly created property.

```xaml
<TextBlock Text="Example English Text"
			uiExtensions:UiExtensions.ChangeLanguage="True" 
			uiExtensions:UiExtensions.TurkishText="Örnek Türkçe Yazı" 
			uiExtensions:UiExtensions.EnglishText="Example English Text"/>
```

When ```ChangeLanguage``` is ```true``` then it will process that ```TextBlock```, if ```false```, then it will skip it.

Finally we need to edit our ```LanguageChanger``` :

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

You can now use your newly-created custom properties to create translations for every ```TextBlock``` in your application purely in XAML.

Do we stop at translating only TextBlocks? Of course not! 

We can use attached properties on every type of string we want to translate. For example, headers, context menus, contents etc. Simply create some attached properties, add them to your ```LanguageChanger```, or whatever class you are using for globalization, and edit your XAML code as we did with TextBlocks.

### Conclusion

Attached properties are a very useful part of WPF, and their use can be extensive on extending or creating new properties for your controls and objects. I found them especially useful when I was attempting globalization. This technique will hopefully help you in your own projects as well.

