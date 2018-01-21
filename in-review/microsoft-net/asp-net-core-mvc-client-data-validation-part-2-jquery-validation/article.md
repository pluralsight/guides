
| Technology | Skill Level |
| :--- | :---
| ASP.NET Core MVC | Beginner
| C# | Beginner
| HTML | Beginner
| JavaScript | Beginner

## Introduction

*This is the second guide in a three part series on client-side data validation in ASP.NET Core MVC web applications.*

Part 1 of this series showed how the Model-View-ViewModel (MVVM) design pattern can enhance the user experience, data validation, and security of ASP.NET Core MVC web apps, focusing on complementary components:

* ViewModels with .NET data annotations and
* Razor Views with HTML5 `<input>` elements.

This installment builds on that foundation by explaining how widely-used jQuery libraries can provide responsive client-side data validation by working in conjunction with the HTML created by ASP.NET Core MVC (hereafter, "MVC") using the code described in Part 1.

Part 3 of the series looks at validating data in controller methods when it is received from an HTML form through an `HttpPost` event.

### Sample code

A Visual Studio 2017 solution containing the complete code for each part of this series is available on [GitHub](https://github.com/ajsaulsberry/BlipValidation).

### Components

Three JavaScript files comprise the jQuery validation functionality of a MVC solution: **jquery.js**, **jquery.validate.js**, and **jquery.validate.unobtrusive.js**. Together, they work with the HTML elements that comprise HTML `<form>` elements to provide *unobtrusive validation* in the client browser. The `data-` HTML attributes that MVC creates and populates as it builds the HTML page are key elements in the unobtrusive validation functionality.

The **jquery.validate.unobtrusive.js** code was originally developed and copyrighted by Microsoft. It has since been open-sourced along with the rest of ASP.NET. The code provides the interactive, invisible until needed, aspect of unobtrusive validation.

The **jquery.validate.unobtrusive.js** code is built on the **jquery.validate.js** library, an open source project. The **jquery.validate.js** library provides the actual validation functionality, including rules, messages, and event handlers.

The **jquery.js** library is a ubiquitous JavaScript library from the [JS Foundation](https://js.foundation/community/projects) that "...makes things like HTML document traversal and manipulation, event handling, animation, and Ajax much simpler with an easy-to-use API that works across a multitude of browsers." Both the validation libraries depend on **jquery.js**, as do portions of the Bootstrap CSS framework.

### Functionality

Unobtrusive validation provides the user with data validation and validation messages as the user enters data into fields on a web page form. The messages appear as the user enters data for each field. Validation messages appear below the associated field in red text, by default, and do not take up any space unless they are rendered. Messages can pertain to a variety of validation criteria:

* required fields,
* string length (minimum and maximum number of characters),
* value range (minimum and maximum values for dates and numbers), and
* allowable characters (matching regular expressions)

The criteria for validation are determined by the field's data type and data attributes as expressed in the model class associated with the Razor view. In MVVM design, this is the viewmodel.

Unobtrusive validation can supply either default messages or custom messages provided as parameters of data annotations. Custom error messages can be literal strings or references to resource files. Messages can be localized to the current culture context.

## Adding jQuery validation to a MVC web project

The **good news** is that including jQuery validation in a web project is very easy in Visual Studio 2017. Using the Visual Studio 2017 tooling to create a new ASP.NET Core Web Application from the Model-View-Controller template will automatically include the following JavaScript libraries in the folder structure shown:

~~~
<solution>/<project>/wwwroot/lib
    bootstrap/dist/js
        **bootstrap.js**
    jquery/dist
        jquery.js
    jquery-validation/dist
        addtional-methods.js
        jquery.validate.js
    jquery-validation-unobtrusive
        jquery.validate.unobtrusive.js
~~~

(Note that wwwroot/js/site.js is provided for code written specifically for the project.)

At a glance, it would appear all the pieces are in place to implement nicely formatted client-side data validation using standard components.

The **bad news** comes in five parts:

1. the included libraries are not current versions;
1. they cannot be updated with the NuGet Package Manager;
1. the recommended way of updating the libraries, [Bower](https://bower.io), has been deprecated;
1. the alternatives are more complex and take time to learn and implement; and
1. there is a shifting landscape of competing and complementary package management tools.

These factors add up to a level of complexity that puts an explanation of the package management required update jQuery and the validation libraries way beyond the scope of this guide. See the [Additional Resources](#additional-resources) section for pointers to more information on package management for ASP.NET Core.

Fortunately, there is nothing about the basic functionality of jQuery validation that requires the latest version of any of the libraries. Make a note that the package management issue needs to be addressed for new MVC projects, and the jQuery libraries updated before being put in production, but put that aside for the purposes of learning to work with the technology.

## jQuery validation libraries

There are three libraries required to implement responsive, client-side data validation in ASP.NET Core web pages. Each of these is installed by the default *New New ASP.NET Core Web Application* template in Visual Studio 2017, but the most recent version is **not** installed. [1]

| Library | Current Version | Installed Version
| :--- | :--- | :---
| jquery.js | 3.2.1 | 2.2.0
| jquery.validate.js | 1.17.0 | 1.14.0
| jquery.validate.unobtrusive.js | 3.2.6 | 3.2.6

### Inconsistencies

There are a number of inconsistencies associated with these files:

1. The jQuery Validation plugin directory is "jquery-validation" and filename is "jquery.*validate*.js. [Emphasis added.]

    1. On content delivery networks (CDN's), the directory for the plugin will be *either* "jquery-validate" or "jquery-validation", depending on the CDN.

    1. The package namespace "jquery-validate" is deprecated. The current package namespace for **jquery.validate.js** is "jquery-validation".

1. The paths under /jquery/ and /jquery-validation/ include a /dist/ folder, but /jquery-validation-unobtrusive/ does not. The /dist/ folders are commonly used by package managers to store the minimized versions of the code files ("dist" = "distribution").

1. Dots ("`.`") are used as separators in "**jquery.validate.js**", but a hyphen ("`-`") is used in "jquery-validation/additional-methods.js".

1. There is no version number in the header for **jquery.validate.unobtrusive.js**, as delivered by the template. There *is* a version number in the bower.json file installed with the code file and it reflects the current version (as of this writing): 3.2.6.

1. The stated copyright for **jquery.validate.unobtrusive.js** for the file delivered by the template is by Microsoft, but the actual copyright in place since Microsoft made ASP.NET an open source project is the [MIT License](https://opensource.org/licenses/MIT). This is reflected in the license.txt file in the source code repository on GitHub: [https://github.com/aspnet/jquery-validation-unobtrusive]. The other components are also offered under the MIT license.

### Dependencies

jQuery is a JavaScript library. JavaScript must be enabled in the browser for unobtrusive validation to work. MVC provides for [graceful degradation](#graceful-degradation) in the event JavaScript is not enabled, providing fallback validation functionality.

The **jquery.validate.js** and **jquery.validate.unobtrusive.js** files are plugins for the jQuery library. Plugins extend the prototype `jquery` object (more commonly referred to in code as `$`) by adding additional methods. The plugins will not work without the jQuery library, so **jquery.js** must load from a CDN or a specified fallback location for unobtrusive validation to work.

The **jquery.validate.unobtrusive.js** plugin depends on the **jquery.validate.js** plugin.

Some functionality for the Bootstrap CSS framework is implemented using jQuery in the library: bootstrap/dist/js/**bootstrap.js**. It is advisable to load the Bootstrap JavaScript library before the validation libraries. Accordingly, the suggested load order is:

1. **jquery.js**
1. **bootstrap.js**
1. **jquery.validate.js**
1. **jquery.validate.unobtrusive.js**

In fact, this is the load order provided for in the template files. Since Bootstrap functionality may potentially be used on any page in the website, **jquery.js** and **bootstrap.js** are loaded in the MVC layout file `<project>/Views/Shared/_Layout.cshtml`. Baring any other changes to the design of the site, the layout file will be rendered in every view.

## Referencing the validation code

Visual Studio tooling and the `dotnet new mvc` CLI command provide convenient ways to initialize an MVC project with widely used settings which make implementation of unobtrusive client-side validation convenient and transparent. While it is convenient to rely on this functionality, it is a good idea to understand how it works.

Developers using [Visual Studio Code](https://code.visualstudio.com/) to build MVC projects have less tooling available and will have to code more of their view (.cshtml) files by hand, so it is critical that they understand what to include, and where to include it, to make unobtrusive validation work.

In a standard view there are five files that contain code necessary for unobtrusive validation to work:

1. **_ViewImports.cshtml**
1. **_ViewStart.cshtml**
1. **_Layout.cshtml**
1. **_ValidationScriptsPartial.cshtml**
1. **_viewname_.cshtml**

By default, every view provided by Visual Studio in the initial MVC template and every view created using the tooling references the following files located in the `<project>/Views` folder:

~~~cmd
_ViewImports.cshtml
_ViewStart.cshtml
~~~

### **_ViewImports.cshtml**

The [**_ViewImports.cshtml**](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/layout#viewimports) file provides for Razor directives and dependency injection across the views within a folder structure. This is important to unobtrusive validation because three standard directives included in this file are required to transparent implementation of jQuery plugins:

~~~csharp
@using AddjQuery.Web
@using AddjQuery.Web.Models
~~~

These directives include the Visual Studio SDK tooling necessary to add the `<script>` references to new Razor views.[2]

~~~csharp
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
~~~

This directive adds [Tag Helper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) functionality to the Razor views. Tag Helpers are used by MVC to generate HTML elements and the `data-*` element attributes on which unobtrusive validation depends.

### **_ViewStart.cshtml**

The **_ViewStart.cshtml** file contains a single Razor directive specifying the Layout to use for views:

~~~csharp
@{
    Layout = "_Layout";
}
~~~

This is important for validation functionality because** _Layout.cshtml** contains the `<script>` elements for loading **jquery.js** and **bootstrap.js** (which improves the presentation aspects of validation).

### **_Layout.cshtml**

The **_Layout.cshtml** file defines HTML, Razor directives, tag helpers, and content common to all the views in a given directory structure that share the same **_ViewStart.cshtml** file. The **_Layout.cshtml** file created by the standard MVC template includes markup to load the **jquery.js** and **bootstrap.js** libraries from the local machine or a content delivery network (CDN), depending on the attributes of the [`<environment>`](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/environment-tag-helper) helper tag.

Additionally, the non-development environment `<script>` elements for these JavaScript libraries include [ScriptTagHelper](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.scripttaghelper?view=aspnetcore-2.0) helper tags to specify:

* a fallback source for the library if the CDN source is unavailable,
* a test to determine if the library has successfully loaded from the CDN, and
* a value for the [subresource integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) check.

By default, it also loads the site's custom JavaScript file, site.js, and appends a cache busting value to the file name in non-development environments.

~~~html
...
    <environment include="Development">
        <script src="~/lib/jquery/dist/**jquery.js**"></script>
        <script src="~/lib/bootstrap/dist/js/**bootstrap.js**"></script>
        <script src="~/js/site.js" asp-append-version="true"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.2.0.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha384-K+ctZQ+LL8q6tP7I94W+qzQsfRV2a+AfHIi9k8z8l9ggpc8X+Ytst4yBo/hH+8Fk">
        </script>
        <script src="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/bootstrap.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa">
        </script>
        <script src="~/js/site.min.js" asp-append-version="true"></script>
    </environment>

    @RenderSection("Scripts", required: false)
</body>
</html>
~~~

### **_ValidationScriptsPartial.cshtml**

This partial view is included by reference by the Visual Studio *Add MVC View* tooling for templates that include `<form>` elements. The file includes markup to load the **jquery.validate.js** and **jquery.validate.unobtrusive.js** libraries. Like the code in **_Layout.cshtml**, the source for the libraries is determined by the current runtime environment.

In the sample code below, note the following:

* The local paths for **jquery.js** and **jquery.validate.js** include a **/dist/** directory, but the path for **jquery.validate.unobtrusive.js** does not.

* The CDN path for **jquery-validation** and **jquery-validation-unobtrusive** point to the [Microsoft Ajax Content Delivery Network](https://docs.microsoft.com/en-us/aspnet/ajax/cdn/overview). It is perfectly valid to use other CDN's such as [CDNJS](https://cdnjs.com/). The paths will vary between CDN's.

* The CDN path for **jquery.validate.js** points to an older version (that corresponds to the version of the code included with the MVC template project).

~~~html
<environment include="Development">
    <script src="~/lib/jquery-validation/dist/**jquery.validate.js**"></script>
    <script src="~/lib/jquery-validation-unobtrusive/**jquery.validate.unobtrusive.js**"></script>
</environment>
<environment exclude="Development">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.14.0/jquery.validate.min.js"
            asp-fallback-src="~/lib/jquery-validation/dist/jquery.validate.min.js"
            asp-fallback-test="window.jQuery && window.jQuery.validator"
            crossorigin="anonymous"
            integrity="sha384-Fnqn3nxp3506LP/7Y3j/25BlWeA3PXTyT1l78LjECcPaKCV12TsZP7yyMxOe/G/k">
    </script>
    <script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.6/jquery.validate.unobtrusive.min.js"
            asp-fallback-src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"
            asp-fallback-test="window.jQuery && window.jQuery.validator && window.jQuery.validator.unobtrusive"
            crossorigin="anonymous"
            integrity="sha384-JrXK+k53HACyavUKOsL+NkmSesD2P+73eDMrbTtTk0h4RmOF8hF8apPlkp26JlyH">
    </script>
</environment>
~~~

Note that the code shown above constitutes the complete contents of the file. Partial views do not contain any HTML `<head>` content or any other markup outside the portion of a complete HTML document they represent.

### Razor views with `<form>` elements

When the Visual Studio tooling is used to create a new view containing a `<form>` element, references to the unobtrusive validation plugins are added automatically. This happens when the view is added through the *Add MVC View* dialog box and the appropriate values are selected for a form type view in the Template and Model fields.

Visual Studio creates the `.cshtml` file for the view and adds the following code to the bottom:

~~~csharp
@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
~~~

The `Scripts` section is rendered in `_Layout.cshtml` after the `<script>` references for **jquery.validate.js** and **bootstrap.js**, so the load order for views containing forms is optimal; the validation libraries follow the jQuery, Bootstrap, and site-wide custom JavaScript (site.js). 

It is important to keep in mind that because this code is in a `@section` the load order is determined by the **_Layout.cshtml** file in which the section is rendered, rather than the view containing the section. If the section is placed at the top of the view the content will still load after the body of the view.

The validation libraries are also loaded asynchronously using the HtmlHelper `Html.RenderPartialAsync`. Because the code is rendered this way some rules regarding [partial views](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/partial) apply.  Loading the libraries this way is intended to improve performance.

## Validation in practice

How does validation look in practice? If all the default elements of unobtrusive validation are implemented correctly, the user interface, generated HTML, and code will look much like the examples shown in the next section. Try out the code and experiment with modifications using the [case study project](#case-study-project).

MVC also provides for *graceful degradation*, which is a way of making the validation process work as consistently and completely as possible if pieces of the implementation are missing or disabled.

### Unobtrusive validation

When successfully implemented, unobtrusive validation provides feedback as the user is typing or when the focus leaves the `<input>` element. The screenshots and code below show what is rendered in the browser before and after data entry and the code from which the rendering is done.

#### /Validation/CustomerAdd rendered before data entry

When the **/Validation/CustomerAdd** web page opens the **Company Name** field on the page looks like this:


![company name field before entry](https://raw.githubusercontent.com/pluralsight/guides/master/images/795b3ac9-2d41-4311-9932-e62a241a8e74.png)

The cursor is in the field and the field is shown as having the focus with a blue shaded border. This is a result of `autofocus="autofocus"` being set in the Razor view.

The field also shows a prompt message specified in the CustomerAdd.cshtml Razor view.

#### /Validation/CustomerAdd HTML before data entry

~~~html
...
<div class="form-group">
    <label class="control-label" for="CompanyName">Company Name</label>
    <input class="form-control" placeholder="Enter your cool business handle."  
        autocomplete="organization" autofocus="autofocus" type="text" 
        data-val="true" data-val-length="The field Company Name must be a string with a minimum length of 5 and a maximum length of 100." 
        data-val-length-max="100" data-val-length-min="5" data-val-required="The Company Name field is required." 
        id="CompanyName" name="CompanyName" value="" />
    <span class="text-danger field-validation-valid" data-valmsg-for="CompanyName" data-valmsg-replace="true"></span>
</div>
...
~~~

The `data-` ("data dash") attributes are created by MVC from the data annotations in the viewmodel associated with the view. For data attributes where no `Message` parameter is specified, such as the `StringLength` and `Required` annotations, MVC creates default messages for the `data-val-length` and `data-val-required` attributes used by the jQuery validation code.

#### CustomerAdd.cshtml

Note that the value for the `placeholder` attribute is specified directly in the Razor view.

~~~html
      <div class="form-group">
        <label asp-for="CompanyName" class="control-label"></label>
        <input asp-for="CompanyName" class="form-control" placeholder="Enter your cool business handle." autocomplete="organization" autofocus="autofocus" />
        <span asp-validation-for="CompanyName" class="text-danger"></span>
      </div>
~~~

The markup in the Razor view is combined with the data type and data annotations of the viewmodel property associated with the field specified in the `asp-for` and `asp-validation-for` attributes of the `<label>`, `<input>`, and `<span>` elements associated with the property.

#### CustomerAddViewModel.cs

Note that the `Prompt` parameter of the `CompanyName` property in the viewmodel is overridden by the `placeholder` attribute of the `<input>` element in the Razor view.

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
...
~~~

In the example above, one might reasonably add a `RegularExpression` data attribute to specify a range of allowable characters for the company name. The regular expression string might be stored in a resource file associated with a specific culture context, as this constraint is country specific.

#### /Validation/CustomerAdd after validation

If the user enters data that does not meet the validation requirements, such as "Bl", shown above, and tabs to the next field, the associated validation message appears below the field. The `<span>` element containing the validation message text did not occupy any space on the rendered page prior to the message appearing.



![company name field after entry](https://raw.githubusercontent.com/pluralsight/guides/master/images/106d9e31-272f-40a5-a4b4-90d0d40cfcab.png)

#### /Validation/CustomerAdd HTML after validation

Using the browser's F12 developer tools it is possible to see the HTML that renders the validation message and the changes to the `data-` attributes made by the jQuery validation code.

~~~html
...
<div class="form-group">
    <label class="control-label" for="CompanyName">Company Name</label>
    <input class="form-control input-validation-error" 
        placeholder="Enter your cool business handle." autocomplete="organization" autofocus="autofocus" type="text" data-val="true" 
        data-val-length="The field Company Name must be a string with a minimum length of 5 and a maximum length of 100." 
        data-val-length-max="100" data-val-length-min="5" 
        data-val-required="The Company Name field is required." id="CompanyName" name="CompanyName" value="" 
        aria-required="true" aria-invalid="true" aria-describedby="CompanyName-error">
        <span class="text-danger field-validation-error" data-valmsg-for="CompanyName" data-valmsg-replace="true">
            <span id="CompanyName-error" class="">The field Company Name must be a string with a minimum length of 5 and a maximum length of 100.</span></span>
</div>
...
~~~

Note that the validation message associated with the `StringLength` data annotation in the viewmodel now appears in the `<span>` element. This text was added by the jQuery validation code and did not require a roundtrip to the server.

### Graceful degradation

MVC provides for circumstances in which unobtrusive validation is missing or not working. It automatically responds to the absence of the various components to maintain a user experience that is as consistent as possible. The fallback levels are as follows:

1. If **jquery.validate.unobtrusive.js**, **jquery.validate.js**, or **jquery.js** (or their minimized and bundled equivalents) are missing or disabled, all validation rules are applied to `<input>` elements and the text appears just as it did for unobtrusive validation, **except** that the validation occurs and messages are displayed after the user submits the form (presses the `<submit>` button) and a roundtrip is made to the server. This happens even if JavaScript is disabled entirely.

2. If **bootstrap.css** is missing or disabled the validation messages are still displayed. The form associated with the examples above would look like this:


![form without bootstrap](https://raw.githubusercontent.com/pluralsight/guides/master/images/86a53c71-e8e7-4a77-aaf7-63d16222d0c6.png)

Note that browser data entry controls will still work and the available ranges will reflect the values set in `min` and `max` attributes.

The most important things to keep in mind about a degraded validation environment is that unless all the elements of unobtrusive validation are working (or jQuery validation code has been specifically written) are:

1. validation will not occur until the form has been submitted and
1. validation will require a roundtrip to the server for each attempt.

User may experience frustration and dissatisfaction if validation requires repeated form submissions and roundtrip latency.

## Conclusion

The collection of technologies that constitute unobtrusive validation provide important enhancements to the user interface in web applications built with ASP.NET Core MVC. Timely and informative information about data validation improves the convenience and efficiency of data entry in addition to reducing data entry errors.

Because validation takes place in the client browser it reduces roundtrips to the server and thereby reduces server workload as well as user interface latency. Good data validation also helps improves security.

Standard Visual Studio tooling makes implementation of unobtrusive validation easy. Developers using Visual Studio Code can implement unobtrusive validation with only a little more effort by following the pattern explained in this guide. Solutions developed this way are interoperable between Visual Studio 2017 and Visual Studio Code.

## Additional resources

Check out the resources below for sample code,  in-depth explanations, reference documentation, video courses, and helpful tools relating to the material covered in this guide.

### Case study project

A Visual Studio 2017 solution containing the complete code from which the examples shown above are drawn is available at:

[github.com/ajsaulsberry/BlipValidation](https://github.com/ajsaulsberry/BlipValidation)

(PluralSight and the author disclaim any liability for the accuracy and completeness of the sample code and make no warranty respecting its suitability for any purpose. The sample code is subject to revision.)

### PluralSight courses

PluralSight provides complete learning paths for each technology covered by this series of guides:

[ASP.NET MVC](https://www.pluralsight.com/paths/mvc5)

[JavaScript](https://www.pluralsight.com/paths/javascript)

[jQuery](https://www.pluralsight.com/paths/jquery)

There are many classes pertaining specifically to ASP.NET Core. Of particular interest to developers getting started with this technology are:

[ASP.NET Core Fundamentals](https://www.pluralsight.com/courses/aspdotnet-core-fundamentals)

[ASP.NET Core MVC Request Life Cycle](https://www.pluralsight.com/courses/aspnet-core-mvc-request-life-cycle)

[ASP.NET Core Tag Helpers](https://www.pluralsight.com/courses/aspdotnet-core-tag-helpers)

PluralSight also offers a number of JavaScript and jQuery classes. Of particular relevance to the topics addressed in this series of guides are the following:

[jQuery Forms and Bootstrap 3](https://www.pluralsight.com/courses/jquery-forms-bootstrap3) See especially Module 4, The jQuery Validation Plugin.

[JavaScript for C# Developers](https://www.pluralsight.com/courses/js4cs) A great course for developers who comfortable with C# but new to JavaScript.

### Web resources

Disclaimer: Neither PluralSight nor the author are responsible for the accuracy or completeness of information provided by 3rd parties, including the resources listed below.

[JavaScript Basics Tutorial](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/JavaScript_basics) - the Mozilla Developer Network provides a basic introduction to JavaScript that is well suited to those who are entirely new to JavaScript.

[learn.jquery.com](http://learn.jquery.com/) - the jQuery Foundation offers a comprehensive tutorial on jQuery, including a section on Ajax (Asynchronous JavaScript And ~~XML~~ JSON).

Microsoft's docs.microsoft.com has documentation for both ASP.NET Core and the .NET API. (The older documentation is still available and linked-to in many places; use the URL's below to get to the new versions.)

[Microsoft Docs: ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/index)

[Microsoft Docs: .NET API Reference - Core 2.0](https://docs.microsoft.com/en-us/dotnet/api/index?view=netcore-2.0)

The Mozilla Developer Network provides excellent guides and reference documentation for HTML, CSS, JavaScript and related technologies:

[MDN web docs](https://developer.mozilla.org/en-US/)

[CodePen](https://codepen.io) provides a nice way to try out HTML, CSS, and JavaScript without spinning up IIS Express and the Visual Studio debug environment.

---

## Notes

1. This is true at the time of this writing, but Microsoft can change the template at any time or alter the way the libraries are implemented.

2. This is speculation based on behavior: there appears to be no documentation for these directives or the namespaces.

---

Written by AJ Saulsberry, 23 January 2018