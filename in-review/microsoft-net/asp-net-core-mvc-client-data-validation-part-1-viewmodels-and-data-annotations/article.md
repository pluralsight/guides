| Technology | Skill Level |
| :--- | :---
| ASP.NET Core MVC | Beginner
| C# | Beginner
| HTML | Beginner

## Introduction

ASP.NET Core MVC can provide powerful client-side data validation through the [Model-View-ViewModel](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel) (MVVM) design pattern and common JavaScript libraries. The key to using these technologies effectively is understanding both the underlying concepts and the features of the technologies.

MVVM provides a way to increase the [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) in a web application by organizing data from the [data access layer](https://en.wikipedia.org/wiki/Data_access_layer) of an application into a form that facilitates interaction by the application's users. In an ASP.NET MVC Core application, this enables the data fields in HTML form elements be bound to a class object consisting of strongly-typed properties.

This guide, the first of three parts, presents the foundational level of client-side data validation using the MVVM design pattern: viewmodels and data annotations. The following articles in this series expand on this foundation as follows:

* Part 2: jQuery validation
* Part 3: Controller validation

### Sample code

A Visual Studio 2017 solution containing the complete code for the examples shown in this guide is available on [GitHub](https://github.com/ajsaulsberry/BlipValidation). *(The author disclaims responsibility for errors or omissions in the sample code.)*

## Benefits of MVVM-based data validation

Using viewmodels in the MVVM design pattern provides a number of specific benefits to an application's user experience, robustness, maintainability, and security.

### Tiered validation

Data validation can be applied to the viewmodel in the presentation layer of the application, the web page running in the web browser on the client machine, and at the server, in the controller method that handles the `HttpPost` action for the associated form. ViewModels can be constructed so that significant data validation happens on the client, especially when jQuery validation libraries are used.

The controller method associated with the `HttpPost` action can leverage the viewmodel to determine if  the model state is valid and appropriately handle cases in which it is not. The controller method can also perform validation that depends on data that cannot be transmitted to the browser during the `HttpGet` action or on logic that cannot (or should not) be implemented in the browser.

How much of the data validation takes place in the client browser and how much is done by the controller method is a design decision that depends on the business logic associated with the form as well as the capabilities of the technologies. Security should always be considered; sensitive information used for validation can be better protected by keeping it on the server.

Performing validation of viewmodel data does not eliminate the requirement that the data access layer validates data before committing it to the data store. In MVC applications the interface between business logic and the [domain model](https://en.wikipedia.org/wiki/Domain_model) and, in turn, the data store is frequently implemented through the use of the [repository pattern](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application). Repository methods should maintain the consistency of data transferred between the data model entities and the data store, often a relational database, by performing appropriate validation.

Validation done in the client web browser and on the server in the controller's `HttpPost` method can both be considered to be part of *client* data validation. Both are intended to ensure that data submitted with the HTML form conforms to the validation requirements for each field and that form collection comprises a valid model, as defined by the viewmodel to which the form is bound.

### Data entry enhancement

Using viewmodels and data annotations has benefits beyond helping users enter the correct data. Doing so can also help make data entry easier and more efficient.

#### User controls

The client data validation process can also assist the user with entering data. Modern browsers include default user interface elements for facilitating entry of common data types such as dates. For example, Google Chrome provides a user interface control to assist users with entering dates, which looks like the following in a desktop browser:


![Date entry control](https://raw.githubusercontent.com/pluralsight/guides/master/images/a1d542d6-3367-4445-8acb-df6d6b97bb16.png)

All that's required to get this functionality is adding the following attribute to the associated property of the data model:

~~~csharp
[DataType(DataType.Date)]
public DateTime TransactionDate { get; set; }
~~~

These type of controls create a better user experience, making it easier for users to enter data quickly and accurately. By providing a guide for data entry they also make it easier for the user to determine the expected format for the data.

#### Alternate input methods

Strongly-typed data entry fields are particularly helpful on mobile devices, which lack physical keyboards. Mobile devices can present users with a specialized virtual keyboard optimized for the associated data type. The following is an example of a virtual keyboard for phone number entry on Android devices:


![phone number keyboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/2ad32c6d-9664-4785-91e0-195c19b3400e.png)

This keyboard appears whenever a mobile web browser detects an `<input>` field with a `type` attribute set to `tel`, as shown in the following example:

`<input id="workphone" type="tel" />`

For some, but not all, HTML5 data types the appropriate HTML is created by ASP.NET based on the data type in the viewmodel. The [ ] section will provide more detail on how that works in MVC.

#### Localization

One common web design consideration is localization. Using an appropriate data type enables the browser to use the appropriate format. For example, in Google Chrome date and number fields are rendered with the language settings defined in the [browser settings](https://support.google.com/chrome/answer/173424?co=GENIE.Platform%3DDesktop&hl=en-GB&oco=1):

| Data Type | French (France) | English (United States)
| :--- | :--- | :---
| Date | 26/11/2017 | 11/26/2017
| Number | 12 345,67 | 12,345.67

### Security

From a security perspective, using appropriate data types for input elements is a good idea. Although other parts of a web application's code may have primary responsibility for preventing [common types of attack](https://en.wikipedia.org/wiki/Web_application_security), such as [cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) (XSS) and [SQL injection](https://en.wikipedia.org/wiki/SQL_injection), keeping input values within the appropriate bounds helps mitigate risk and helps keep the application working as expected.

* data type,
* format,
* string length, and
* numeric range

A well built viewmodel can help prevent malicious users from sending "too much" data. For example, an entity representing a transaction may have a property for an "approval date". Binding a `<form>` element to a viewmodel that excludes the approval date property helps prevent a malicious user from using submitting an unauthorized value for this property as part of an HttpPost response for a form that is not bound to a strongly-typed model.

For example:

~~~csharp
// Transaction.cshtml
// The view may not include all the properties of a transaction, or may include them as hidden elements.
@model BlipValidate.Data.Entities.Transaction
...

// TransactionController.cs
// Vulnerable to an overposting attack: model can include Approved property even if the form does not.
[HttpPost]
public IActionResult CustomerAdd(Object model)
{  ...  }

// Transaction.cshtml
// The view may not include all the properties of a transaction, or include them as hidden elements.
@model BlipValidate.Data.ViewModels.PurchaseViewModel
...

// Strongly typed. The viewmodel can exclude the Approved DateTime property in the Transaction entity.
[HttpPost]
public IActionResult CustomerAdd(PurchaseViewModel model)
{  ...  }
~~~

Some data types, such as credit card numbers, impose more implementation requirements than common data types, such as text and integers. These requirements include security-related constraints intended to ensure confidential information is handled appropriately. Part 3 of this series of guides will look at those data types in more detail.

## ViewModels and Views

ViewModels provide separation of concerns in ASP.NET Core MVC, enabling the organization of data for presentation and interaction to be abstracted from the properties of entities in the domain data model. ViewModels may be structured around an application's use cases, while the domain data model may be a collection of entities that reflects the relational structure of an underlying database.

ASP.NET Core MVC tooling uses viewmodels to automatically create standardized views for create, delete, inspect (details), edit, and list operations. While these standard templates may be too limited for a number of interaction scenarios, they can sometimes be used as the basis for creating a more complex form or partial view, eliminating a portion of the coding that would otherwise be required.

### Creating views from viewmodels

[ASP.NET Core MVC](https://docs.microsoft.com/en-us/aspnet/core/mvc/overview) takes care of mapping C# value types and `string` to the appropriate HTML `type` attribute when the `asp-for` [Tag Helper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) is used with the `<input>` HTML element. Using tag helpers can improve the consistency and reliability of an application's HTML by automating repetitive HTML coding required to create form elements.

Using Visual Studio tooling to create views can further reduce the coding necessary to create an MVC view when viewmodels are used. Specifying a viewmodel in the **Model Class** field of the **Add MVC View** dialog box will automatically create an appropriate view for a number of standard interactions.

#### **Add MVC View** dialog box


![Add MVC View dialog box](https://raw.githubusercontent.com/pluralsight/guides/master/images/bd2fdeb4-a1cb-42f0-b7fb-7e4db3eb5729.png)

The dialog box above will create a new view, **AddCustomer.cshtml**, using the properties of the **CustomerAddViewModel.cs** class. The new view will include an HTML `form` element with appropriate tag helpers for:

* the `<form>` element;
* a validation summary element works in conjunction with the `ModelState` validation of the controller;
* `<label>`, `<input>`, and `<span>` (validation message) elements for each property of the viewmodel;
* an `<input>` of type `submit` for the save/submit button associated with the form; and
* Bootstrap CSS classes appropriate to each of the preceding elements.

#### CustomerAdd.cshtml view created by Visual Studio tooling

Ellipsis ("`...`") in sample code indicates sections of code redacted for brevity. For complete code, see the accompanying example project, [BlipValidation](https://github.com/ajsaulsberry/BlipValidation), on GitHub.

~~~html
<div class="row">
  <div class="col-md-4">
    <form asp-action="CustomerAdd">
      <div asp-validation-summary="ModelOnly" class="text-danger"></div>
      <div class="form-group">
        <label asp-for="CompanyName" class="control-label"></label>
        <input asp-for="CompanyName" class="form-control" />
        <span asp-validation-for="CompanyName" class="text-danger"></span>
      </div>
      ...
      <div class="form-group">
        <input type="submit" value="Save" class="btn btn-default" />
      </div>
    </form>
  </div>
</div>
~~~

The HTML generated by ASP.NET Core MVC at runtime will include the preceding elements and:

* the `<form>` element's `action` and HTTP `method` attributes;
* [HTML5 data attributes](#html5-data-attributes) (`data-*`) for use by JavaScript validation libraries;
* validation messages defined in the viewmodel (or default messages);
* an `<input>` element containing a ASP.NET Core MVC [Request Verification Token](https://docs.microsoft.com/en-us/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages); and
* values and validation messages determined at runtime in response to user input.

#### CustomerAdd.cshtml rendered in the browser

~~~html
<div class="row">
  <div class="col-md-4">
    <form action="/Validation/CustomerAddBare" method="post">
      <div class="form-group">
        <label class="control-label" for="CompanyName">CompanyName</label>
        <input class="form-control" type="text" id="CompanyName" name="CompanyName" value="" />
        <span class="text-danger field-validation-valid" data-valmsg-for="CompanyName" data-valmsg-replace="true"></span>
      </div>
      ...
      <div class="form-group">
        <input type="submit" value="Save" class="btn btn-default" />
      </div>
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8POOhkwigahBv3LUQr7vwTXZAgaEnft386uviEvEOCQjIKHat5IxezOlK3YrtNM-HN3x5Rw52jwF6sNkbJs_2RjoFlMdyoZ2eZOERXMKl68EuxfbefbhmDzuTakVRXsfYbvQ8njfLGXrt3WXeMGOfSQ" /><input name="Nonprofit" type="hidden" value="false" /></form>
  </div>
</div>
~~~

Note that when the view is first rendered in response to an HttpGet it does not include the validation summary element because there are no validation errors at that point. When the form is submitted to the controller through an HttpPost action (when the user clicks the **Save** button) it's model state can be evaluated using the `ModelState` object and the view re-rendered with the validation summary if the model state is invalid. The validation state element will then be rendered along with text values indicating the errors for each `<input>` element.

### Type translation

Many types of data presented and collected in HTML forms conform to C# [value types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/value-types), the [string](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/string) reference type, and the [System.DateType](https://docs.microsoft.com/en-us/dotnet/api/system.datetime?view=netcore-2.0) Struct. But the range of HTML5 `<input>` element `type` attributes to which these types can be mapped is limited.

It is important to understand that ASP.NET Core MVC handles the conversion between these data types and HTML5 element types. When the form is rendered in serialized HTML and sent to the browser in response to an HttpGet action the viewmodel properties are rendered as `<input>` elements according as shown in the table below (if no [data annotations](#using-data-annotations-with-viewmodels) are applied to the property). In response to form data submitted through an HttpPost action, MVC converts the serialized data to the associated type specified in the viewmodel and constructs the model object.

As the table below shows, some numeric types are converted to text elements. Since [decimal](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/decimal) values are used frequently in applications dealing with money and the decimal data type is one that is converted to a text element, the type conversion for decimal value information is important to a broad range of applications.

| C# Value Type | HTML Type Attribute |
| :--- | :--- |
| bool | checkbox *
| byte | number
| char | text
| decimal | text
| double | text
| float | text
| int | number
| long | number
| sbyte | number
| short | number
| uint | number
| ulong | number
| ushort | number

| C# Reference Type | HTML Type Attribute |
| :--- | :--- |
| string | text

| System Class | HTML Type Attribute |
| :--- | :--- |
| DateTime | datetime-local* (also date, time, with data annotations)

&ast; The HTML input type `datetime` [is obsolete](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/datetime). Use `datetime-local` instead; it provides culture-specific presentation.

### Bootstrap CSS classes

ASP.NET Core MVC creates a `<div>` element of the Bootstrap CSS class `.form-group` to surround each set of elements related to each `<input>` element. Each form group typically contains the following elements:

* `<label>`,
* `<input>`,
* `<span>` (for validation error messages)

Each element in a form group is also automatically assigned a CSS class appropriate to the type of element. Some input types, such as **checkbox** and **dropdown** will have additional `<div>` elements within the form group and additional styling rules.

In the [Bootstrap](https://getbootstrap.com/docs/3.3/) CSS, the related form group elements are automatically given formatting rules to create clean, consistent forms. Using [Bootstrap Themes](https://themes.getbootstrap.com/) and project-specific customization of Bootstrap classes, developers can further refine the presentation of form elements.

### HTML5 data attributes

To enable form elements to work in conjunction with JavaScript to provide client-side data validation, ASP.NET Core MVC generates [HTML5 data attributes](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes) known as *custom data attributes* that enable information to be exchanged between the HTML elements and the JavaScript validation methods. The examples below shows typical code for the portion of a viewmodel and the view associated with a decimal field used to enter a stock price. There are custom data attributes for the valid input type (number) and the minimum (1) and maximum (5,000) values (sufficient for every stock except [BRK.A](https://www.google.com/search?q=NYSE:BRK.A)).

#### CustomerAddViewModel.cs

~~~csharp
...
[Display(Name = "Max. Stock Price")]
[Range(1, 5000)]
public decimal? StockPriceMax { get; set; }
...
~~~

#### CustomerAdd.cshtml

~~~html
...
<div class="form-group">
  <label class="control-label" for="StockPriceMax">Max. Stock Price</label>
  <input class="form-control" type="text" data-val="true" data-val-number="The field Max. Stock Price must be a number." data-val-range="The field Max. Stock Price must be between 1 and 5000." data-val-range-max="5000" data-val-range-min="1" id="StockPriceMax" name="StockPriceMax" value="" />
  <span class="text-danger field-validation-valid" data-valmsg-for="StockPriceMax" data-valmsg-replace="true"></span>
</div>
...
~~~

### Nullable value types

In the example above, a value is not required for the **Max. Stock Price field**, therefore the `StockPriceMax` property in the view model is defined as a [nullable type](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/nullable-types/) through the use of the `T?` syntax : `decimal?`. If the viewmodel required a value for `StockPriceMax` the HTML generated by ASP.NET Core MVC for the `<input>` element `id="StockPriceMax"` would also include an additional custom data attribute:

~~~html
data-val-required="The Max. Stock Price field is required."
~~~

Nullable types are particularly important when working with data in a relational database where datatypes corresponding to C# value types, such `bool` may have a `null` value. For example, in SQL Server Boolean values are represented by a bit which can be null as well as 0 or 1:

| C# Type | C# Value | SQL Server Value
| :--- | :--- | :---
| **boo**l | | |
| | true | 1
| | [not supported] | null
| | false | 0
| **bool?** | | |
| | true | 1
| | null | null
| | false | 0

Nullable value types in C# can be very helpful when dealing with existing database record sets which have incomplete data or data entry scenarios where the user cannot be expected to have all the necessary information during a single session. The latter case is also one where the abstraction provided by MVVM is useful: a value may be optional (nullable) in viewmodels representing certain stages of data collection, but ultimately required when committing the data to the database via the entity classes in the data model. By expressing these requirements in the viewmodel, views that depend on it and controller actions that evaluate model state can ensure the associated business logic is consistently applied.

## Using Data Annotations with ViewModels

Data annotations provide a way of describing the data represented by a viewmodel class in a way that is more specific than the datatype of the properties alone can convey. This information frequently incorporates business rules such as required data, minimum and maximum values, allowable characters, and level of precision.

Incorporating these rules into the viewmodel separates the rules about the data from the presentation of the data, which is handled by the view. This requires less code in the view and leverages ASP.NET Core MVC's ability to combine the rules in the viewmodel with the presentation defined in the view to generate an HTML page that implements both in the browser.

Relying on the automation provided by ASP.NET Core MVC helps ensure consistency in the way HTML elements are rendered. In particular, using tag helpers enables MVC to automatically create HTML custom data attributes that implement the validation rules defined with the viewmodel's data annotations. Aside from providing consistency, this also eliminates a lot of anodyne coding.

Data Annotations are part of the `System.ComponentModel.DataAnnotations` namespace, so viewmodels must include a corresponding `using` statement:

~~~csharp
using System.ComponentModel.DataAnnotations;
~~~

The [System.ComponentModel.DataAnnotations](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.datatype?view=netcore-2.0) namespace provides a number of classes that enable:

* data to be described according to standard or custom patterns;
* actions, such as comparisons and validation, to be performed on the data in model classes.

Data is described in classes such as the [RangeAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.rangeattribute?view=netcore-2.0), [RequiredAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.requiredattribute?view=netcore-2.0), and [StringLengthAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.stringlengthattribute?view=netcore-2.0). In code, application of these data annotations looks like this:

~~~csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace BlipValidate.Data.ViewModels
{
    public class CustomerAddViewModel
    {
        [Display(Name = "Company Name")]
        [StringLength(100, MinimumLength = 5)]
        [Required(ErrorMessage = "A company name is required when creating a customer.")]
        public string CompanyName { get; set; }
        ...
        [Display(Name = "Max. Stock Price")]
        [Range(1, 5000)]
        public decimal? StockPriceMax { get; set; }
    }
}
~~~

In this example the `CompanyName` property is required. Since the type of the property is the nullable reference type `string`, the `[Required]` attribute is necessary. (But the attribute's error message is not: MVC will generate an error message in the HTML and implement the appropriate client-side validation with jQuery validation.) Because the `StockPriceMax` property is nullable (`decimal?`), a value is not required for the model state to be valid.

### The `DataTypeAttribute` data annotation

In the preceding example a number of data annotations are used to provide information about the data in the viewmodel, but the datatypes themselves, `string` and `decimal?` are not modified. In the example, these standard types are sufficient.

In some cases a greater degree of specificity or customization is necessary. This can be done with the `DataTypeAttribute` class, or classes that inherit from it.

* When a greater degree of specificity is required, the **DataTypeAttribute** can be used in conjunction with the [DataType Enumeration](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.datatype?view=netcore-2.0). The `DataType` *enumeration*, when used in the `DataTypeAttribute`, and applied to a viewmodel, is used by MVC to generate more specific `type` attributes for the `<input>` elements of HTML generated from views. It also enables MVC to chose the most appropriate template for creating the HTML `<input>` type itself.

* The enumeration values for the `DataType` annotation **do not provide any validation**, nor do they alter the underlying data type of the viewmodel property.

* When customization is required, classes that inherit from **DataTypeAttribute** can be used to create object classes that take the place of the standard types that are modified by the **DataType** enumeration. (This is a more advanced topic and is beyond the scope of this guide. For more information, follow the links in the table below for each DataTypeAttribute class.)

The table below lists the available DataType enumeration values, the corresponding DataTypeAttribute class inheritors (where they exist), and the property type associated with the enumeration.

| DataType Enumeration | Annotation<br />DataTypeAttribute&nbsp;class | Property Type | Description |
| :--- | :--- | :--- | :--- |
| CreditCard | CreditCard<br />[CreditCardAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.creditcardattribute?view=netcore-2.0) | string<br />class | Represents a credit card number.
| Currency | | decimal | Represents a currency value.
| Custom | | class | Represents a custom data type.
| Date | | DateTime | Represents a date value.
| DateTime | | DateTime | Represents an instant in time, expressed as a date and time of day.
| Duration | | DateTime | Represents a continuous time during which an object exists.
| EmailAddress| EmailAddress<br />[EmailAddressAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute?view=netcore-2.0) | string<br />class | Represents an e-mail address.
| Html | | string | Represents an HTML file.
| ImageUrl | | string | Represents a URL to an image.
| MultilineText | | string | Represents multi-line text.
| Password | | string | Represent a password value.
| PhoneNumber | Phone<br />[PhoneAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.phoneattribute?view=netcore-2.0) | string<br />class | Represents a phone number value.
| PostalCode | | string | Represents a postal code.
| Text | | string | Represents text that is displayed.
| Time |  | DateTime | Represents a time value.
| Upload | | string | Represents file upload data type.
| Url | Url<br />[UrlAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.urlattribute?view=netcore-2.0) | string<br />class | Represents a URL value.

It is a good idea to use the `DataTypeAttribute` with the `DataType` enumeration, rather than one of the classes that derives from `DataTypeAttribute` (for example; `CreditCard`, `Email`) unless the decorated property is an object.

~~~csharp
// Use this.
[DataType(DataType.PhoneNumber)]
public string MobilePhone { get; set; }

// Not this.
[Phone]
public string MobilePhone { get; set; }
~~~

### Mapping DataType enumerations to HTML type attributes

When MVC combines the viewmodel with the view and creates the HTML, the `type` attributes for the `<input>` elements are set as shown in the following table:

| DataType Enumeration | HTML Type Attribute |
| :--- | :---
| CreditCard | text
| Currency | text
| Date | date
| DateTime | datetime-local
| Duration | datetime-local
| Email | email
| Html | text
| ImageUrl | text
| MultilineText | text
| Password | password
| PhoneNumber | tel
| PostalCode | text
| Text | text
| Time | time
| Upload | text
| Url | url

Note the way data types relating to dates and times are mapped. Also be aware that a property designated `DataType.CreditCard` should only be implemented in web projects for which SSL/TLS has been implemented.

Getting the `<input>` element `type` attributes right is important because they govern the way the browser handles the data types on different platforms (mobile, desktop). Using the appropriate data type annotation can help ensure this happens.

### Validation attributes

There are many member classes of the `System.ComponentModel.DataAnnotations` namespace, and the member classes are used in a wide variety of contexts. Nine member classes inherit from the [ValidationAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute?view=netcore-2.0) class,  including `DataTypeAttribute`. Of the `ValidationAttribute` classes there are four that are the most relevant to data validation in viewmodels:

1. [Range](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.rangeattribute?view=netcore-2.0) - sets the minimum and maximum allowable values for a number, as expressed as an `Int32`. Use requires setting hardcoded values. `Range` does not work with jQuery validation when setting minimum and maximum values for a `DateTime` property. (Hardcoding dates is also to be avoided.)

1. [RegularExpression](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.regularexpressionattribute?view=netcore-2.0) - works with jQuery validation to implement client-side validation according to a regular expression.

1. [Required](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.requiredattribute?view=netcore-2.0) - used with nullable data types (usually `string` or `DateTime`).

1. [StringLength](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.stringlengthattribute?view=netcore-2.0) - can be used to set minimum and maximum length. (`StringLength` is customarily used for viewmodels; the `MinLength` and `MaxLength` validation attributes are used for entity models.) The `minimumLength` parameter can be used to create a length range for strings.

### HTML5 `<input>` element attributes

As noted [above](#mapping-datatype-enumerations-to-HTML-type-attributes), MVC maps .NET data types to the closest corresponding HTML `<input>` element `type` attribute. MVC will also create `<input>` element `type` attributes to implement the constraints expressed in data annotations associated with properties in the viewmodel.

The validation and user experience of the web page can be further refined by adding `type` attributes to the `<input>` elements in the view and assigning values to them with Razor code. There are also `type` attributes that alter the behavior of the form in ways that can improve the user experience and facilitate faster, more accurate data entry. Some of these attributes are [global](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes), while others are specific to individual input types.

* `autocomplete` will enable the user to select from previously entered values for similar fields that have been stored by the browser.

* `autofocus` sets the focus to the designated `input` element when the page loads. Chrome and other browsers show the field has the focus with a visual clue. Nice from the user's perspective: they can start typing at the first field on the form, rather than having to tab or mouse to it to get started with data entry.

* `min` and  `max` attributes can be used in conjunction with data from the viewmodel to set the range for input values. This is particularly helpful with dates because the `Range` data attribute is incompatible with jQuery validation.

* `pattern` (`date`, `datetime-local` and `time` types) is used to set the input pattern (e.g. "dd/mm/yyyy") for data and time fields. This is useful when the desired input pattern is different than the pattern derived automatically from the browser's localization context.

* `pattern` (`email`, `password`, `search`, `tel`, `text`, `url` types) takes the form of a regular expression that defines the allowable string. Note that the `RegularExpression` data annotation creates a `data-val-regex-pattern` attribute that works with jQuery validation, `pattern` works without JavaScript and thereby provides graceful degradation to text-only browser environments.

* `placeholder` can be used to provide an example of the desired input or an input tip in the input element. This element can be created with the `Display` data annotation when the `Prompt` parameter is used. When the `placeholder` attribute exists in the view (*.cshtml) and in a `Display` data annotation with a `Prompt` parameter exists in the viewmodel, the value of `placeholder` in the view takes precedence.

~~~csharp
[Display(Name = "User Name", Prompt = "Enter a user name of between 5 and 20 characters.")]
[StringLength(20, MinimumLength = 5)]
[Required]
public string Username { get; set; }
~~~

### Case study code

The following code snippets show the viewmodel, controller actions, view, and generated HTML for a web page that demonstrates data annotations and `<input>` element `type` attributes. The code can be found in the example project on [GitHub](https://github.com/ajsaulsberry/BlipValidation).

Ellipsis ("`...`") in sample code indicates sections of code redacted for brevity.

#### CustomerAddViewModel.cs (with more properties and data annotations)

~~~csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace BlipValidate.Data.ViewModels
{
    public class CustomerAddViewModel
    {
        [Display(Name = "Company Name", Prompt = "Registered name of the company")]
        [StringLength(100, MinimumLength = 5)]
        [Required]
        public string CompanyName { get; set; }

        [Display(Name = "HQ Country Code")]
        [StringLength(3, MinimumLength = 3)]
        [RegularExpression(@"[A-Za-z]{3}", ErrorMessage = @"The country code must be three letters.")]
        [Required]
        public string HqCountryIso3 { get; set; }

        [Display(Name = "Number of Employees")]
        [Range(1, 2500000)]
        public int EmployeeCount { get; set; }

        [Display(Name = "Incorporated")]
        [DataType(DataType.Date)]
        [Required]
        public DateTime? Incorporated { get; set; }

        public bool Nonprofit { get; set; }

        [Display(Name = "Max. Stock Price")]
        [Range(1, 5000)]
        public decimal? StockPriceMax { get; set; }

        [Display(Name = "Audit Date")]
        [DataType(DataType.Date)]
        // Nullable data type required to prevent initialization from creating
        // a default value of 01/01/0001 00:00:00.
        public DateTime? AuditDate { get; set; }

        public DateTime EarliestAudit { get; set; }

        public DateTime LatestAudit { get; set; }
    }
}
~~~

#### ValidationController.cs

~~~csharp
using System;
using Microsoft.AspNetCore.Mvc;
using BlipValidate.Data.ViewModels;

namespace BlipValidate.Web.Controllers
{
  public class ValidationController : Controller
  {
      ...
      public IActionResult CustomerAdd()
      {
          var model = new CustomerAddViewModel
          {
              EarliestAudit = DateTime.Parse("2017-04-11"),
              LatestAudit = DateTime.Parse("2017-04-29")
          };
          return View(model);
      }

      [HttpPost]
      [ValidateAntiForgeryToken]
      public IActionResult CustomerAdd(CustomerAddViewModel model)
      {
          if(ModelState.IsValid)
          {
              return RedirectToAction("Index");
          }
          else return View();
      }
    ...
  }
}
~~~

#### CustomerAdd.cshtml view

Note that the dates for the `min` and `max` attributes of the `AuditDate` `<input>` element have to be converted to `yyyy-MM-dd` strings to be in the format expected by the browser. For more information on formatting date and time strings, see docs.microsoft.com on [Custom Date and Time Format Strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings).

~~~html
@model CustomerAddViewModel
...

<div class="row">
    <div class="col-md-4">
      <form asp-action="CustomerAdd" id="customerAdd">
        <div asp-validation-summary="ModelOnly" class="text-danger"></div>
        <div class="form-group">
          <label asp-for="CompanyName" class="control-label"></label>
          <input asp-for="CompanyName" class="form-control" placeholder="Enter your cool business handle." />
          <span asp-validation-for="CompanyName" class="text-danger"></span>
        </div>
        ...
        <div class="form-group">
          <div class="checkbox">
            <label>
              <input asp-for="Nonprofit" /> @Html.DisplayNameFor(model => model.Nonprofit)
            </label>
          </div>
        </div>
        <div class="form-group">
          <label asp-for="StockPriceMax" class="control-label"></label>
          <input asp-for="StockPriceMax" class="form-control" />
          <span asp-validation-for="StockPriceMax" class="text-danger"></span>
        </div>
        <div class="form-group">
          <label asp-for="AuditDate" class="control-label"></label>
          <input asp-for="AuditDate" class="form-control" min=@Model.EarliestAudit.ToString("yyyy-MM-dd") max=@Model.LatestAudit.ToString("yyyy-MM-dd") />
          <span asp-validation-for="AuditDate" class="text-danger"></span>
        </div>
        <div class="form-group">
          <input type="submit" value="Save" class="btn btn-default" />
        </div>
      </form>
    </div>
</div>
...
~~~

#### CustomerAdd.cshtml rendered HTML

Note that the rendered HTML shown below is redacted to correspond to the view code shown above.

~~~html
...
<div class="row">
    <div class="col-md-4">
      <form id="customerAdd" action="/Validation/CustomerAdd" method="post">
        ...
        <div class="form-group">
          <label class="control-label" for="CompanyName">Company Name</label>
          <input class="form-control" placeholder="Enter your cool business handle." type="text" data-val="true" data-val-length="The field Company Name must be a string with a minimum length of 5 and a maximum length of 100." data-val-length-max="100" data-val-length-min="5" data-val-required="The Company Name field is required." id="CompanyName" name="CompanyName" value="" />
          <span class="text-danger field-validation-valid" data-valmsg-for="CompanyName" data-valmsg-replace="true"></span>
        </div>
        ...
        <div class="form-group">
          <div class="checkbox">
            <label>
              <input type="checkbox" data-val="true" data-val-required="The Nonprofit field is required." id="Nonprofit" name="Nonprofit" value="true" /> Nonprofit
            </label>
          </div>
        </div>
        <div class="form-group">
          <label class="control-label" for="StockPriceMax">Max. Stock Price</label>
          <input class="form-control" type="text" data-val="true" data-val-number="The field Max. Stock Price must be a number." data-val-range="The field Max. Stock Price must be between 1 and 5000." data-val-range-max="5000" data-val-range-min="1" id="StockPriceMax" name="StockPriceMax" value="" />
          <span class="text-danger field-validation-valid" data-valmsg-for="StockPriceMax" data-valmsg-replace="true"></span>
        </div>
        <div class="form-group">
          <label class="control-label" for="AuditDate">Audit Date</label>
          <input class="form-control" min="2017-04-11" max="2017-04-29" type="date" id="AuditDate" name="AuditDate" value="" />
          <span class="text-danger field-validation-valid" data-valmsg-for="AuditDate" data-valmsg-replace="true"></span>
        </div>
        <div class="form-group">
          <input type="submit" value="Save" class="btn btn-default" />
        </div>
      <input name="__RequestVerificationToken" type="hidden" value="CfDJ8POOhkwigahBv3LUQr7vwTWLyRdGXzzvPLn93yRXqTeQ9TuRC60aoQtv9ZWb_kBGE5xiN4Z0QudyVpxHsqlWPCquxsq1mXfQ_bdmN4RutsqOWEwARlH9-3F5S-Qm23VsYrXyMk0exjvLlv8y4Zqe3AA" /><input name="Nonprofit" type="hidden" value="false" /></form>
    </div>
</div>
...
~~~

#### Rendered web page /Validation/CustomerAdd (in Chrome)

Note that the browser uses the `min` and `max` attributes of the `<input` element `AuditDate` to format the data entry control to show the valid date range.

![Customer Add page](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf27ac6c-daa2-4a06-9c7a-0d4b3f3addc6.png)

## Conclusion

The Model-View-ViewModel (MVVM) design pattern improves ASP.NET Core MVC projects. When carefully designed, viewmodels and views add a number of benefits:

* improved separation of concerns;
* a better user experience, with
  * browser controls,
  * alternate input methods,
  * unobtrusive and responsive field and form level validation; and
* better security.

Correct use of classes from the `System.ComponentModel.DataAnnotations` namespace and `<input>` element attributes is essential to getting the most out of viewmodels and views. While there are many classes in the `DataAnnotations` namespace, mastering the ones relevant to client-side data validation is, for the most part, straightforward.

## Additional resources

Check out the resources below for sample code,  in-depth explanations, reference documentation, video courses, and helpful tools relating to the material covered in this guide.

### Case study project

A Visual Studio 2017 solution containing the complete code from which the examples shown above are drawn is available at:

[github.com/ajsaulsberry/BlipValidation](https://github.com/ajsaulsberry/BlipValidation)

(PluralSight and the author disclaim any liability for the accuracy and completeness of the sample code and make no warranty respecting its suitability for any purpose. The sample code is subject to revision.)

### PluralSight courses

PluralSight provides an entire learning path for ASP.NET MVC technology, with courses by some of the industry's leading experts:

[ASP.NET MVC](https://www.pluralsight.com/paths/mvc5)

There are many classes pertaining specifically to ASP.NET Core. Of particular interest to developers getting started with this technology are:

[ASP.NET Core Fundamentals](https://www.pluralsight.com/courses/aspdotnet-core-fundamentals)

[ASP.NET Core MVC Request Life Cycle](https://www.pluralsight.com/courses/aspnet-core-mvc-request-life-cycle)

[ASP.NET Core Tag Helpers](https://www.pluralsight.com/courses/aspdotnet-core-tag-helpers)

### Web resources

Note: Neither PluralSight nor the author are responsible for the accuracy or completeness of information provided 3rd parties, including the resources listed below.

Microsoft's docs.microsoft.com has documentation for both ASP.NET Core and the .NET API. (The older documentation is still available and linked-to in many places; use the URL's below to get to the new versions.)

[Microsoft Docs: ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/index)

[Microsoft Docs: .NET API Reference - Core 2.0](https://docs.microsoft.com/en-us/dotnet/api/index?view=netcore-2.0)

The Mozilla Developer Network provides excellent guides and reference documentation for HTML, CSS, and related technologies:

[MDN web docs](https://developer.mozilla.org/en-US/)

[RegExr](https://regexr.com/) is an excellent tool for verifying that your regular expressions mean what you think they do.

[CodePen](https://codepen.io) provides a nice way to try out HTML, CSS, without spinning up IIS Express and the Visual Studio debug environment.

---

Written by AJ Saulsberry, 15 January 2018