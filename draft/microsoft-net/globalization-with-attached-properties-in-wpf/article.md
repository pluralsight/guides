Recently during my PhD studies, as a part of the thesis project, I needed an application that works on Windows so I decided to go for a WPF application. I developed the application but there was a problem. My thesis was supposed to be in English so the application was supposed to be English; however the place where the application was going to be used was a local place, hence it had to be Turkish as well. So I had to do some globalization for my application to be both English and Turkish.

There are many ways to do globalization, such as setting up string resources, having custom static classes that hold the original and translated text or using some globalization libraries but I wanted something simple and easy on the XAML and the code behind like:

```xaml
<TextBlock Text=”English Text” English=”English Text” Turkish=”Türkçe Yazı” />
```

After a bit of research I came upon a concept called “attached properties” which can be used to set custom properties on XAML.

### What are attached properties?

According to an MSDN article; “An attached property is a concept defined by Extensible Application Markup Language (XAML). An attached property is intended to be used as a type of global property that is settable on any object. In Windows Presentation Foundation (WPF), attached properties are typically defined as a specialized form of dependency property that does not have the conventional property wrapper”.
So basically they are the existing properties that we use in everyday WPF development such as:

```xaml
<DockPanel>
   <CheckBox DockPanel.Dock="Top">Hello</CheckBox>
</DockPanel>
```

Going over the example in the MSDN article; Dockpanel.Dock is an attached property that is predefined for us.

So using attached properties you can create new custom properties that suit your needs.

### Attached Properties in Globalization

After finding out about attached properties, my purpose was to create attached properties that will automatically select the text, header or tooltip that matches the current language of the application.

To get started let's first deal with TextBlocks. Attached properties work like extension methods, here is the code to add translated text to a ```TextBlock``` :

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
It is fairly simple; we create 2 seperate properties for both languages named TurkishText and EnglishText. These properties will be of type ```string``` . Finally we define the getters and setters for these properties.

So now that we defined the properties, we can add them to our XAML code.

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

