Recently, as a part of my phD thesis project, I needed an application that works on Windows. I decided to go for a Windows Presentation Foundation (WPF) application, which I ended up developing. But there was a problem. 

My thesis was supposed to be in English so the application was supposed to be English. Yet the application was going to be used locally, and had to be in Turkish as well. So I had to globalize my application to be both in English and Turkish.

There are many ways to perform globalization, such as setting up string resources, having custom static classes that hold the original and translated text or using some globalization libraries. But I wanted something simple and easily done with Extensible Application Markup Language (XAML). The code would look like:

```xaml
<TextBlock Text=”English Text” English=”English Text” Turkish=”Türkçe Yazı” />
```

After a bit of research I came upon a concept called “attached properties” which can be used to set custom properties on XAML.

### What are attached properties?

According to an [MSDN article](https://msdn.microsoft.com/en-us/library/ms749011%28v=vs.100%29.aspx); “An attached property is a concept defined by Extensible Application Markup Language (XAML). An attached property is intended to be used as a type of global property that is settable on any object. In Windows Presentation Foundation (WPF), attached properties are typically defined as a specialized form of dependency property that does not have the conventional property wrapper”.
So basically they are the existing properties that we use in everyday WPF development such as:

```xaml
<DockPanel>
   <CheckBox DockPanel.Dock="Top">Hello</CheckBox>
</DockPanel>
```

Going over the example in the MSDN article; Dockpanel.Dock is an attached property that is predefined for us.

Using attached properties, you can create new custom properties that suit your needs.

### Attached Properties in Globalization

After finding out about attached properties, I aimed to create attached properties that would automatically select the ```Text```, ```Header```, or ```Tooltip``` that matches the current language of the application.

As an example I will show how you can use attached properties with TextBlocks. Attached properties work like extension methods, here is the code to add translated text to a ```TextBlock``` :

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
The code above is fairly simple. First, we create 2 separate properties for both languages, which are named ```TurkishText``` and ```EnglishText```. These properties will be of type ```string``` . Then we define the getters and setters for these properties.

Now that we defined the properties, we can add them to our XAML code.

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
So now we can use our newly created extension like this:

```xaml
<TextBlock Text="Example English Text" 
			uiExtensions:UiExtensions.TurkishText="Örnek Türkçe Yazı" 
			uiExtensions:UiExtensions.EnglishText="Example English Text"/>
```

However, running the application, we see that the text does not actually change. We still need to tell the application that we want to change the language somehow.

We can simply write a ```LanguageChanger``` class and a few helper classes to do this job for us.

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

The LanguageChanger code should allow our WPF to find all the TextBlocks and change their language to whichever language setting we decide. 

However, there is a big problem here. Our application will try and change **ALL** TextBlocks in our XAML. So if it cannot find any ```EnglishText``` or ```TurkishText```, the application will default to empty strings which are not very desirable because we do not want **ALL** our TextBlocks to change, just the blocks that are ```EnglishText```.

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

