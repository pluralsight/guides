| Technology | Skill Level |
| :--- | :---
| ASP.NET Core MVC | Beginner
| C# | Beginner
| HTML | Beginner

## Introduction

*This is the third guide in a three part series on client-side data validation in ASP.NET Core MVC web applications.*

[Part 1](https://www.pluralsight.com/guides/microsoft-net/asp-net-core-mvc-client-data-validation-part-1-viewmodels-and-data-annotations) of this series showed how the Model-View-ViewModel (MVVM) design pattern can enhance the user experience, data validation, and security of ASP.NET Core MVC web apps, focusing on complementary components:

* ViewModels with .NET data annotations and
* Razor Views with HTML5 `<input>` elements.

[Part 2](https://www.pluralsight.com/guides/microsoft-net/asp-net-core-mvc-client-data-validation-part-2-jquery-validation) builds on that foundation by explaining how widely-used jQuery libraries can provide responsive client-side data validation by working in conjunction with the HTML created by ASP.NET Core MVC (hereafter, "MVC") using the code described in Part 1.

### Model binding and data validation

MVC [model binding](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding) provides powerful capabilities for handling the data sent during an HTTP POST. This guide looks at how two parts of this functionality play a role in client data validation:

* Data model binding takes form data submitted through the HTTP POST method as key value pairs and parses it into an instance of a type (object) passed as a parameter to a controller action method. In MVVM design, the data model parameter type is the viewmodel combined with the Razor view to create the form.

* The `ModelState` class provides a convenient way of 1) determining if data comprising a valid model has been returned by the client and then 2) responding to errors at the model or individual property level.

### Other functionality

It is worth noting that MVC model binding also handles two other parts of the data sent by the browser:

1. route information and
1. query strings.

It is possible to customize how model binding handles data in the HTTP request by using data in the HTTP header, binding the request data to parameters of application services through [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection), and overriding the default binding functionality.

These features are well worth further exploration, but they are beyond the scope of this guide. To learn more, continue your self-guided discovery at [PluralSight.com/guides](https://www.pluralsight.com/guides).

### Sample code

Examples in this guide are based on the same case study used in the previous two parts. A Visual Studio 2017 solution containing the complete code for each part of this series is available on [GitHub](https://github.com/ajsaulsberry/BlipValidation).

Ellipsis (`...`) in the sample code shown below indicates portions of the code redacted for brevity. See the case study solution for the complete code.

## Client data validation in MVC controllers

Data validation performed in controller action methods can be considered to be part of the client data validation process. While the validation occurs on the server, it is concerned with collecting data accurately from the end user working with the client-side portion of the application that is running in the browser.

In many cases the validation done in the controller method is indistinguishable to the end user from the validation done by jQuery unobtrusive validation in the client browser: it uses the same data types, data annotations, and messages to perform the validation. As explained in Part 2, data validation performed on the server is a transparent fallback in circumstances where jQuery unobtrusive validation (or custom jQuery validation) is missing or disabled.

### Form data security

The fundamental step in form security is to [enforce SSL](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl). Implementing SSL/TLS for the entire site is the recommended practice and is actively [encouraged by search engines](https://searchengineland.com/google-emails-warnings-webmasters-chrome-will-mark-http-pages-forms-not-secure-280907). The Chrome browser will [display a warning](https://blog.chromium.org/2017/04/next-steps-toward-more-connection.html) on pages with `<form>` elements that do not implement HTTPS with a valid security certificate.

MVC also provides convenient tools for helping to prevent cross-site request forgeries (XSRF or CSRF). MVC automatically generates an antiforgery request token for `<form>` elements for HTTP POST actions. The HTML shown below is an example of the antiforgery request token `<input>` element included near the end of a `<form>` element.

#### /Validation/CustomerAdd rendered HTML

~~~html
...
<input name="__RequestVerificationToken" type="hidden" value="CfDJ8POOhkwigahBv3LUQr7vwTWLyRdGXzzvPLn93yRXqTeQ9TuRC60aoQtv9ZWb_kBGE5xiN4Z0QudyVpxHsqlWPCquxsq1mXfQ_bdmN4RutsqOWEwARlH9-3F5S-Qm23VsYrXyMk0exjvLlv8y4Zqe3AA" />
...
~~~

The request token is verified at the controller action method by using the `[ValidateAntiForgeryToken]` attribute on the view's `HttpPost` action method:

#### /Controllers/ValidationController.cs

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult CustomerAdd(CustomerAddViewModel model)
{
...
~~~

If the verification token passed by the form is missing or invalid, as is likely to be the case with a fraudulent request, the POST request is rejected.

Using strongly-typed models with Razor views, as is the practice in MVVM design, also helps with form security by providing a model object that can be more precisely evaluated for correct and complete data.

### Data model binding

As shown in Part 1 of this series, a viewmodel and a Razor view can be combined using the MVVM design pattern to create a robust form in an HTML webpage. Together, they enable automated generation of HTML elements  incorporating data validation, user interface enhancements, and security elements.

Using a form from the case study project it is possible to see how data model binding works to contribute to client data validation. The **CustomerAdd** view collects a few pieces of data about a hypothetical new customer when a new record is created for the customer.

#### /Validation/CustomerAdd


![validation customer add page](https://raw.githubusercontent.com/pluralsight/guides/master/images/e48e1627-aa6b-43a3-ad6e-08ba0b1327b6.png)

#### Use strongly-typed models

The most important step in performing data validation at the controller is to use strongly-typed models in the Razor view and as the data model parameter type for the controller action method. The viewmodel for the **CustomerAdd** view is shown below.

#### /BlipValidate.Data/ViewModels/CustomerAddViewModel.cs

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
        [Required]
        // Nullable required to prevent initialization from creating a default value of 01/01/0001 00:00:00.
        public DateTime? AuditDate { get; set; }

        // If this isn't nullable ModelState.IsValid will be false when the form is submitted.
        // Since the value isn't being stored in the form in an <input type="hidden" /> it won't be returned to the controller.
        // Rather than make it take a needless roundtrip, make it nullable so that when the form is submitted the ModelState will be valid without it.
        public DateTime? EarliestAudit { get; set; }

        // Nullable for the same reason as EarliestAudit.
        public DateTime? LatestAudit { get; set; }
    }
}
~~~

Visual Studio tooling can use the class **CustomerAddViewModel.cs** to create standard Razor view using templates. The **/Validation/CustomerAdd** view was created using the *Edit* template. This saves a lot of work creating the Razor view, but the viewmodel can be just as effectively bound to the view's controls by manually creating the appropriate [tag helpers](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro).

### Value type conversion

It is important to be aware that as MVC generates the HTML for a view at runtime it converts the data types in the viewmodel to HTML data types. Due to the limitations of HTML data types, there is only one numeric type, `number` and many of the C#/.NET numeric types are converted to HTML `text`.

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

When the HTTP POST request for a `<form>` is received by the controller action method MVC converts the data back into the types specified in the viewmodel (if the viewmodel is used as the type for the form data). The resulting object looks like the following when viewed in the Visual Studio property inspector:


![visual studio property inspector](https://raw.githubusercontent.com/pluralsight/guides/master/images/02de3257-abb1-4ee5-b521-728057f5d22d.png)

### Out-of-range values

It is important to note that if jQuery data validation is not done in the client browser, value types returned to the controller will be parsed as 0 (zero) if they are out of range or invalid. This is because value types are inherently required and inherently non-nullable. If a value type is declared as a nullable value type (for example, `sbyte?`) and an out-of-range value is returned, the value will be parsed as `null`. A type `char` value that is out of range (e.g., more than one character) will be returned as 0 (zero).


![visual studio property inspector for value types](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd030ca3-b334-41a9-9665-887652c556cf.png)

This underscores the importance of doing client-side validation in the browser. Without looking at the form data, the controller action method has no way of knowing whether a zero or null value for a value type is the result of invalid (out-of-range) data, no data (the user skipped the field for a nullable value type), or an entry of "0" for the value.

To learn a technique for ensuring the controller has the best available information on the numeric values sent from an HTML form, see the guide: "ASP.NET Core MVC: Building robust numeric fields for HTML forms".

## Responding to the ModelState

Receiving a valid collection of form data at the controller is likely if the essential components of client-side validation have been implemented correctly and communication between the client browser and the web server has not been disrupted by bad actors or technical problems. As detailed in Parts 1 and 2 of this series of guides, the five components of effective client-side validation are:

1. well-designed views using appropriate tag helpers;
1. strongly-typed viewmodels with constraints applied with data annotations;
1. use of additional HTML element attributes where appropriate;
1. correct implementation of the unobtrusive validation components
    1. jquery.js,
    1. jquery.validate.js,
    1. jquery.validate.unobtrusive.js; and
1. controller action methods with strongly-typed form data parameters.

But application security and robust performance will be better if the controller action method can handle a missing or invalid set of form data. Fortunately, ASP.NET Core MVC provides a method of conveniently determining if the incoming data model is valid and identifying invalid fields.

### Null models

It is possible for a controller action method to receive a null model even when a complete set of form data has been entered at the client. This can happen when MVC cannot parse the form data into the structure supplied in the viewmodel.

* One of the most frequently occurring instances of parsing failures leading to null data models occurs in development when the structure of the viewmodel does not provide for correct binding of hierarchical forms. For more information on building the right viewmodel structure and Razor view, see the guide: [ASP.NET MVC - Getting default data binding right for hierarchical views](https://www.pluralsight.com/guides/microsoft-net/asp-net-mvc-getting-default-data-binding-right-for-hierarchical-views).

Since viewmodels are classes, which are nullable types, the first thing a controller action will do is check for a null value for the form data parameter. One way of doing this is demonstrated in the case study solution:

#### /Controllers/ValidationController.cs null model check

~~~csharp
...
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult CustomerAdd(CustomerAddViewModel model)
{
    if (model == null)
    {
        throw new ArgumentNullException(nameof(model));
    }
...
~~~

Aside from throwing an exception, the controller might log the error, populate a new instance of the viewmodel, and turning the client to the same page with a model validation error. The following code, from the `testing-add-modelstate-error` branch of the case study solution repository, shows an example of adding a `ModelError` to the `ModelState`:

#### /Controllers/ValidationController.cs with AddModelError

~~~csharp
...
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult CustomerAdd(CustomerAddViewModel model)
{
    // Force model to be null for testing purposes.
    model = null;

    if (model == null)
    {
        ModelState.AddModelError(string.Empty, "There was a problem saving the data. Please try again.");
        // Re-initialize model.
        ...
        return View(model);
    }
...
~~~

Adding the model state error causes the following to ensue:

1. Since adding a model error with `ModelState.AddError` automatically sets the `ModelState` to  `invalid`, so there is no need for separate code to change the `ModelState`.

1. The invalid `ModelState` triggers MVC to add the HTML markup for the `asp-validation-summary` helper tag in the Razor view.

1. When the page is refreshed the model error will appear at the top of the form (using the default Razor template view).

#### /Validation/CustomerAdd with ModelState error


![model error validation message](https://raw.githubusercontent.com/pluralsight/guides/master/images/4d83ac1a-548b-4c78-8d08-ba5daa8a7d43.png)

To place the `ModelState` error messages on the page, MVC inserts the markup associated with the `asp-validation-summary` helper tag in the Razor view. The code in the Razor view is shown below:

#### CustomerAdd.cshtml

~~~html
...
<div class="row">
  <div class="col-md-5">
    <form asp-action="CustomerAdd" id="customerAdd">
      <div asp-validation-summary="ModelOnly" class="text-danger"></div>
      <div class="form-group">
        <label asp-for="CompanyName" class="control-label"></label>
        <input asp-for="CompanyName" class="form-control" placeholder="Enter your cool business handle." autocomplete="organization" autofocus="autofocus" />
        <span asp-validation-for="CompanyName" class="text-danger"></span>
      </div>
...
~~~

On the client side, the generated HTML includes a `<div>` element with two CSS classes:

1. `text-danger`, a Bootstrap class which turns the text red, and
1. `validation-summary-errors` an undefined class.

The `validation-summary-errors` class can be used in the **/wwwroot/css/site.css** file to add further styling to the validation summary errors.

~~~html
...
<div class="row">
  <div class="col-md-5">
    <form id="customerAdd" action="/Validation/CustomerAdd" method="post">
      <div class="text-danger validation-summary-errors">
        <ul>
          <li>There was a problem saving the data. Please try again.</li>
        </ul>
      </div>
      <div class="form-group">
        <label class="control-label" for="CompanyName">Company Name</label>
        <input class="form-control" placeholder="Enter your cool business handle." autocomplete="organization" autofocus="autofocus" type="text" data-val="true" data-val-length="The field Company Name must be a string with a minimum length of 5 and a maximum length of 100." data-val-length-max="100" data-val-length-min="5" data-val-required="The Company Name field is required." id="CompanyName" name="CompanyName" value="BlipCo" />
        <span class="text-danger field-validation-valid" data-valmsg-for="CompanyName" data-valmsg-replace="true"></span>
      </div>
...
~~~

*The HTML above has been formatted for clarity. Expect the generated HTML not to be as consistently indented.*

It is worth noting that MVC will not insert the validation summary HTML block if `ModelState.IsValid == true`. Accordingly, this HTML will only appear after a roundtrip to the server during which `ModelState.IsValid == false`. This is one of a number of  instances where MVC generates HTML conditionally for helper tags in the Razor view.

### ModelState field errors

In addition to providing a basis for evaluating if the entire model state is valid, the `ModelState` object also contains information about the validation state of each field. This information includes the value that MVC attempted to convert into the data type specified for the viewmodel's property, which can be helpful in responding to the error.

Field-level errors can be included in the validation summary shown on the HTML page by changing the attribute of the `asp-validation-summary` tag helper from `ModelOnly` to `All`. The field-level error messages are displayed in addition to the error messages associated with the individual fields.

The code below, from the `testing-modelstate-field-errors` branch of the case study solution on GitHub, demonstrates how the individual fields in the `ModelState` can be examined and acted upon:

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult CustomerAdd(CustomerAddViewModel model)
{
  ...
    if (ModelState.IsValid)
    {
        return RedirectToAction("Index");
    }
    else
    {
        foreach (var v in ModelState.Values)
        {
            foreach (var e in v.Errors)
            {
                Debug.WriteLine(e.ErrorMessage.ToString());
                ModelState.AddModelError(string.Empty, $@"Looks like the value {v.RawValue.ToString()} doesn't match the required data type.");
            }
        }
        ...
}
~~~

If "Dave" is entered for the Max. Stock Price field the `Debug` output from the above code would be:

~~~cmd
The value 'Dave' is not valid for Max. Stock Price.
~~~

When the web page is refreshed it looks like:


![CustomerAdd view with validation messages](https://raw.githubusercontent.com/pluralsight/guides/master/images/717c650b-127e-4293-b0b1-ba5ef3d24182.png)

Both the model level and field level validation messages are shown when the page is refreshed after the roundtrip to the server.

## Regular expressions

Regular expression can be used to validate input on the client, server, or both. But using regular expressions can be problematic.

### Implementation in viewmodels

A `RegularExpression` data annotation similar to the following could be used to ensure data is properly entered in fields holding `decimal`, `double` and other C# value types mapped to HTML `<input>` elements of `type="text"` . Regular expressions set here can be used by both jQuery unobtrusive validation in the client browser and MVC model validation on the server.

#### CustomerAddViewModel.cs with regular expression

~~~csharp
...
  [Display(Name = "Max. Stock Price")]
  [Range(1, 5000)]
  [RegularExpression(@"^((\d+(\.\d{0,2})?)|(\.\d{0,2}))$", ErrorMessage = "The stock price must be positive a number")]
  public decimal? StockPriceMax { get; set; }
...
~~~

This regular expression will not have any impact on the client unless jQuery unobtrusive validation is implemented. When unobtrusive validation *is* implemented the regular expression is superfluous because the validation code will check for a valid numeric input even though it is being stored as text. On the server, model binding will identify the problem with mismatched input and set the `ModelState` accordingly.

### Implementation in Razor views

Alternately, the regular expression could be applied directly to the `<input>` element in the Razor view using the HTML5 `pattern` attribute with the `<input>` element.

#### CustomerAdd.cshtml view with regular expression

~~~html
...
<div class="form-group">
  <label asp-for="StockPriceMax" class="control-label"></label>
  <input asp-for="StockPriceMax" class="form-control" pattern="^((\d+(\.\d{0,2})?)|(\.\d{0,2}))$"/>
  <span asp-validation-for="StockPriceMax" class="text-danger"></span>
</div>
...
~~~

The `pattern` attribute will be applied without requiring jQuery validation, but it has some **disadvantages**:

* it is only available for client-side validation;

* it is not supported on older browser versions;

* there is no corresponding message attribute (Chrome displays "Please match the requested format." without identifying the pattern); and

* using it will prevent form submission when the input does not match the pattern, so it inhibits server-side validation.

The potential **advantages** of using the `pattern` attribute are that:

* it can coexist with the `data-val-regex-pattern` attribute created by MVC from a `RegularExpression` data attribute in the viewmodel for the form (but it does not make use of the `data-val-regex` message created from the data attribute's `ErrorMessage` parameter) and

* when jQuery unobtrusive validation is enabled viewmodel validation data attributes take precedence, so `pattern` can serve as a backup when jQuery validation is disabled (although the server-based implementation of the data attribute validation is more robust).

An `<input>` element with both validation data attributes on the viewmodel and the `pattern` HTML5 attribute looks like the following.

#### /Validation/CustomerAdd rendered HTML with mixed attributes

~~~html
<input class="form-control valid" pattern="^((\d+(\.\d{0,2})?)|(\.\d{0,2}))$" type="text"
    data-val="true" data-val-number="The field Max. Stock Price must be a number."
    data-val-range="The field Max. Stock Price must be between 1 and 5000."
    data-val-range-max="5000" data-val-range-min="1"
    data-val-regex="The stock price must be positive a number"
    data-val-regex-pattern="^((\d+(\.\d{0,2})?)|(\.\d{0,2}))$" 
    id="StockPriceMax" name="StockPriceMax" value="" aria-invalid="false"
    aria-describedby="StockPriceMax-error" />
~~~

The order of application by jQuery unobtrusive validation is as follows:

1. `data-val-regex-pattern` attribute and the `data-val-regex` message are applied first if the user enters data that does not match the pattern;

1. when the input matches the pattern the `data-val-range-max` and `data-val-range-min` attributes and the `data-val-range` message are applied;

1. the `data-val-number` attribute is not applied, since conforming the the previous two criteria guarantee the third.

Using viewmodel data attributes excluding `RegularExpression` and **not** using the `pattern` HTML5 attribute gives the most consistent user experience, regardless of whether jQuery unobtrusive validation is working or not.

### Regular expression drawbacks

The problem with using a regular expression, regardless of where it is implemented, is that it may not be appropriate to the current culture. In some cultural contexts a period/dot (char 46) is used to separate the fractional portion of a number and in others a comma (char 44) is used. Setting the allowable character pattern in a regular expression without considering cultural context can cause considerable client consternation. (This is the "inverse C7 dilemma".)

It is also possible to inadvertently create a regular expression that uses an inordinate amount of computing power. This can make the application seem unresponsive to a single user and, when run on the server, make a considerable impact on performance.

Regular expressions can be rather opaque. Getting them right can be a challenge and it can be time and energy intensive to determine what a regular expression does, which impacts the maintainability of the code that depends on it.

### Recommendations

1. Follow the [Best Practices for Regular Expressions in .NET](https://docs.microsoft.com/en-us/dotnet/standard/base-types/best-practices) and avoid using regular expressions unless absolutely necessary.

1. When absolutely needed, implement regular expressions in viewmodels using the `RegularExpression` data attribute, rather than as HTML `<input>` element `pattern` attributes. Data attributes are available to both client-side jQuery unobtrusive validation and the server-side MVC processes, while the `pattern` attribute is only available to the client-side JavaScript.

### Conclusion

Server-side code in MVC controllers can play an important role in determining the validity of client data and in responding to defects with individual errors or with the model state collectively. The key ingredients of effective validation in the controller action method are:

* strongly-typed and annotated viewmodels,
* effective use of ASP.NET Core helper tags in Razor views, and
* strongly-typed form data parameters for the action method.

Validation in the controller also helps improve application security by mitigating certain types of attacks.

Using the `ModelState` object effectively enables the controller to conveniently communicate errors to the client and to respond flexibly based on the nature or errors.

Because controller validation leverages the same underlying code as client-side validation, the controller code can focus on responding to errors that cannot be caught effectively at the client, or that should not be handled at that level for security reasons.

Working together, MVC client-side validation and controller validation provide a robust way of improving the user experience, ensuring the integrity of data, and enforcing security with a minimum of code.

## Additional resources

Check out the resources below for sample code,  in-depth explanations, reference documentation, video courses, and helpful tools relating to the material covered in this guide.

### Case study project

A Visual Studio 2017 solution containing the complete code from which the examples shown above are drawn is available at:

[github.com/ajsaulsberry/BlipValidation](https://github.com/ajsaulsberry/BlipValidation)

PluralSight and the author disclaim any liability for the accuracy and completeness of the sample code and make no warranty respecting its suitability for any purpose. The sample code is subject to revision.

### PluralSight courses

PluralSight provides complete learning paths for each technology covered by this series of guides:

[ASP.NET MVC](https://www.pluralsight.com/paths/mvc5)

[JavaScript](https://www.pluralsight.com/paths/javascript)

[jQuery](https://www.pluralsight.com/paths/jquery)

There are many classes pertaining specifically to ASP.NET Core. Of particular interest to developers getting started with the material covered in this guide are:

[ASP.NET Core Fundamentals](https://www.pluralsight.com/courses/aspdotnet-core-fundamentals)

[ASP.NET Core MVC Request Life Cycle](https://www.pluralsight.com/courses/aspnet-core-mvc-request-life-cycle)

### Web resources

Disclaimer: Neither PluralSight nor the author are responsible for the accuracy or completeness of information provided by 3rd parties, including the resources listed below.

Microsoft's ASP.NET Core documentation offers two thoroughly helpful sections on controller validation:

[Model Binding](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding) - includes instructions for customizing model binding behavior.

[Introduction to model validation in ASP.NET Core MVC](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation) - includes instructions for setting up client-side validation with dynamic (Ajax) forms.

Microsoft's docs.microsoft.com has documentation for both ASP.NET Core and the .NET API. (The older documentation is still available and linked-to in many places; use the URL's below to get to the new versions.)

[Microsoft Docs: ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/index)

[Microsoft Docs: .NET API Reference - Core 2.0](https://docs.microsoft.com/en-us/dotnet/api/index?view=netcore-2.0)

---

Written by AJ Saulsberry, 26 January 2018