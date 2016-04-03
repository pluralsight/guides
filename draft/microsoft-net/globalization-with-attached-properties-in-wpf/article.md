Recently during my PhD studies, as a part of the thesis project, I needed an application that works on Windows so I decided to go for a WPF application. I developed the application but there was a problem. My thesis was supposed to be in English so the application was supposed to be English; however the place where the application was going to be used was a local place, hence it had to be Turkish as well. So I had to do some globalization for my application to be both English and Turkish.

There are many ways to do globalization, such as setting up string resources, having custom static classes that hold the original and translated text or using some globalization libraries but I wanted something simple and easy on the XAML and the code behind like:

```xml
<TextBlock Text=”English Text” English=”English Text” Turkish=”Türkçe Yazı” />
```

After a bit of research I came upon a concept called “attached properties” which can be used to set custom properties on XAML.

### What are attached properties?

According to an MSDN article; “An attached property is a concept defined by Extensible Application Markup Language (XAML). An attached property is intended to be used as a type of global property that is settable on any object. In Windows Presentation Foundation (WPF), attached properties are typically defined as a specialized form of dependency property that does not have the conventional property wrapper”.
So basically they are the existing properties that we use in everyday WPF development such as:

```xml
<DockPanel>
   <CheckBox DockPanel.Dock="Top">Hello</CheckBox>
</DockPanel>
```

Going over the example in the MSDN article; Dockpanel.Dock is an attached property that is predefined for us.

So using attached properties you can create new custom properties that suit your needs.

### Attached Properties in Globalization

After finding out about attached properties, my purpose was to create attached properties that will automatically select the text, header or tooltip that matches the current language of the application.
