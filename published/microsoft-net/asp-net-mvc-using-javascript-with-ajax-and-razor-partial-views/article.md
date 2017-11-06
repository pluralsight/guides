## Introduction

Ajax helper methods and extensions in the System.Web.Mvc and System.Web.Mvc.Ajax namespaces can be combined with JavaScript and MVC partial views to create flexible interactive web pages with minimal code. When using these resources, developers should be aware of a few techniques necessary to create effective code.

This guide shows how to effectively implement JavaScript functionality when creating a web page from Razor partial views, including `<form>` elements created using the `Ajax.BeginForm` helper method. This guide is a companion to [ASP.NET MVC - Using Ajax helpers with Razor partial views](https://www.pluralsight.com/guides/microsoft-net/asp-net-mvc-using-ajax-helpers-with-razor-partial-views).

The codes in this guide are derived from the same Visual Studio solution used for the companion guide, available on [GitHub](https://github.com/ajsaulsberry/BipAjax). You can download and run the project to see the techniques illustrated here in action and to experiment on your own.

## Some Details

The case study is a multi-project Visual Studio 2017 solution developed from the default ASP.NET .NET Framework MVC template. It uses Entity Framework 6.1 and the repository and Model View ViewModel (MVVM) design patterns.

The solution consists of three projects that constitute the different layers of the application:

| Project | Application Layer |
| :--- | :--- |
| Blip.Data | Data Context and Repositories |
| Blip.Entities | Data Entities |
| Blip.Web | User Interface (views) and Business Logic (controllers) |

The case study application, **BlipAjax** is a simple system for gathering, storing, and retrieving  geographic and other information about customers. It's not production-ready from either the design or coding perspectives; it exists to illustrate the concepts discussed in this guide. For a complete description of the example solution, see [the companion guide](https://www.pluralsight.com/guides/microsoft-net/asp-net-mvc-using-ajax-helpers-with-razor-partial-views).

In summary, The **Customer/Edit** page consists of a parent view and three partial views. The third partial view can be either a partial view to create a new e-mail address or a partial view to create a new postal address, depending on the value selected in the second partial view. The following illustration shows **Customer/Edit** with annotations identifying the (partial) view, view model, and the controller action (method) associated with rendering the view.

![Customer Edit Annotated](https://raw.githubusercontent.com/pluralsight/guides/master/images/922fbabe-bb44-4ffb-8598-31556f76e543.png)

On the **Customer/Edit** page JavaScript functions are used to populate the **State/Region** dropdown list with the values that correspond with the value selected in the **Country** dropdown. As the user changes the selected country the values for the *State/Region** field must be reset.

## JavaScript and Ajax partial views

There are two aspects to implementation of JavaScript with Ajax partial Razor views:

1. JavaScript libraries that provide the Ajax functionality needed by the ASP.NET Ajax helper methods and

1. implementation-specific scripts that provide client-side functionality.

In the example solution the implementation-specific scripts are those that populate the **State/Region** dropdown values.

## JavaScript libraries

Successful implementation of Ajax partial views depends on properly loading a number of JavaScript libraries.

### jQuery and other site-wide libraries

The default ASP.NET MVC template includes a standard _Layout.cshtml file that implements navigator features and includes references to standard JavaScript libraries. Because _Layout.cshtml is included in every view in the web project, it is a good place to include script references that implement functionality for page elements, such as the `navbar` found included on every page.

Most of the JavaScript libraries are rendered at the bottom of the `<body>` element of _Layout.cshtml using MVC bundling functionality. The only bundle loaded in the `<head>` element is version 2.8.3 of the Modernizr library, a dated version of Modernizr included for development purposes. The Modernizr reference included in _Layout.cshtml should *not* be included in production code. Including script library references and scripts at the end of the page improves page rendering performance.

The user interface functionality provided by the **Bootstrap** CSS framework is included in the **Bootstrap.js** library loaded with the **Bootstrap** bundle. **Bootstrap.js** depends on the ubiquitous **jQuery** library, which is loaded using the **jquery** bundle.

The definition of the **jquery** bundle enables the current version of jQuery to be loaded without changing the bundle definition or _Layout.cshtml to match the current version number. This way, the jQuery library can be updated through the associated NuGet package without updating the code to keep the version number in sync.

Ajax functionality depends on the **jQuery** library, but not the **Bootstrap** library. If the web project does not implement the Bootstrap CSS framework the **Bootstrap** library is unnecessary. In the example solution, BlipAjax, the Bootstrap CSS framework is used to provide user interface styling and functionality for a number of user interface elements, including form elements like the **Country** and **State/Region** dropdown lists.

#### _Layout.cshtml

~~~html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - My ASP.NET Application</title>
    @Styles.Render("~/Content/css")
    @Scripts.Render("~/bundles/modernizr")
    @RenderSection("header", required: false)
</head>
<body>
    <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li>@Html.ActionLink("Home", "Index", "Home")</li>
                    <li>@Html.ActionLink("About", "About", "Home")</li>
                    <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
                </ul>
            </div>
        </div>
    </div>
    <div class="container body-content">
        @RenderBody()
        <hr />
        <footer>
            <p>&copy; @DateTime.Now.Year - My ASP.NET Application</p>
        </footer>
    </div>

    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/bootstrap")
    @RenderSection("scripts", required: false)
</body>
</html>
~~~

Note that the _Layout.cshtml file also includes a `@RenderSection` Razor directive for the `"scripts"` section. This causes any additional JavaScript library references and scripts to be loaded after the jQuery bundle. The order of these elements is important to getting Ajax functionality to work, and the load order is an area where developers have frequently experienced difficulty.

### jquery-validate.js, jquery-validate-unobtrusive.js, and jquery-unobtrusive-ajax.js

Pages that include `<form>` elements, such as **Customer/Edit** can implement client-side data validation using additional jQuery libraries and, optionally, features of the Bootstrap framework for styling. The `jqueryval` bundle loads both of the required libraries:

1. **jquery-validate.js**

1. **jquery-validate-unobtrusive.js**

These libraries are loaded through the **jqueryval** bundle.

Pages that include Ajax functionality, such as **Customer/Edit** need to include an additional JavaScript library:

**jquery-unobtrusive-ajax.js**

This library is loaded through the **jqueryunobtrusive** bundle.

*Note that the standard naming convention for the libraries creates the potential for confusion:* the **jqueryunobtrusive** bundle does **not** include the library for unobtrusive Ajax and the **jqueryunobtrusive** bundle does not include the library for unobtrusive validation.

As shown below, the parent view for **Customer/Edit** includes a `@section` directive for the `scripts` section specified in _Layout.cshtml. The bundles noted above are included in this section and, following the order of the directives in _Layout.cshtml, load the libraries indicated above after loading the **jquery** bundle. This order is essential to proper implementation of the Ajax JavaScript library; it depends on jQuery.

The load order is also important to proper implementation of the validation libraries. Both **jquery-validate.js** and **jquery-validate-unobtrusive.js** require the jQuery library to be loaded first.

The **Edit.cshtml** parent view also includes page-specific JavaScript code which will be described in detail in the next section.

#### Customer/Edit.cshtml

~~~html
@*Models are in partial views.*@

@section header {

}

@{
    ViewBag.Title = "Edit Customer Information";
}

<div class="row">
    <div class="col-md-12">
        <h2>Edit Customer Information</h2>
    </div>
</div>

<div id="EditCustomerPartial">
    @Html.Action("EditCustomerPartial", new { id = Url.RequestContext.RouteData.Values["id"] })
</div>

<div id="SelectAddressTypePartial">
    @Html.Action("AddressTypePartial", new { id = Url.RequestContext.RouteData.Values["id"] })
</div>

<div id="CreateAddress"></div>

<div class="row">
    <div class="form-group">
        <div class="col-md-12">
            <hr />
            <div>
                @Html.ActionLink("Back to List", "Index")
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @Scripts.Render("~/bundles/jqueryunobtrusive") @*For unobtrusive-ajax*@
    @Scripts.Render("~/bundles/jqueryval") @*For validate and validate-unobtrusive*@

    <script type="text/javascript">
    $(document).ready(function () {
        var selectedCountry = $("#Country").val();
        var selectedRegion = $("#Region").val();
        var regionsSelect = $('#Region');
        AddRegions(selectedCountry, regionsSelect);
        if (selectedRegion != null && selectedRegion != '') {
            regionsSelect.val = selectedRegion;
        };
    });

    $("#Country").change(function () {
        var selectedCountry = $("#Country").val();
        var regionsSelect = $('#Region');
        regionsSelect.empty();
        if (selectedCountry != null && selectedCountry != '') {
            AddRegions(selectedCountry, regionsSelect);
        }
    });

    function AddRegions(selectedCountry, regionsSelect) {
        $.getJSON('@Url.Action("GetRegions")', { iso3: selectedCountry }, function (regions) {
            if (regions != null && !jQuery.isEmptyObject(regions)) {
                regionsSelect.append($('<option/>', {
                    value: null,
                    text: ""
                }));
                $.each(regions, function (index, region) {
                    regionsSelect.append($('<option/>', {
                        value: region.Value,
                        text: region.Text
                    }));
                });
            };
        });
    }
    </script>
}
~~~

Note that server-side `@*comments*@` are included with the `@Scripts.Render` directives for the validation and Ajax bundles to eliminate any confusion that might arise over which scripts are included in the bundles as a result of the standard naming convention.

Also note that the order of these directives is not mandatory, but since the Ajax functionality is more critical to the proper functioning of the page than the client-side validation, it is a good idea to load that library immediately after the jQuery library to minimize the likelihood that load errors would prevent the Ajax library from loading. This is particularly important when adding the complexity of loading JavaScript libraries from content delivery networks (CDN's) in production with a local version version of the library as a fallback when the CDN is unavailable.

Finally, note that the `<form>` elements of the **Customer/Edit** page are included in the partial views, rather than the parent **Edit.cshtml** view. Nevertheless, the convention is for JavaScript associated with a partial view to be located in the **.cshtml** file for the *parent* view, rather than the *partial* view to which they apply. The next section will show this convention in practice as well as describing a notable exception.

## Implementation-specific scripts

The **Customer/Edit** page includes implementation-specific JavaScript functions that populate the values for the **Country** and **State/Region** dropdown lists in both the *Edit customer details* and *Add a new postal address* sections of the page. As noted above, the coding convention in ASP.NET MVC is to put the scripts for a partial view in the **.cshtml** file for the parent view. (There is one important exception to this convention, which will be described in detail below.)

### Implementation of **CustomerEditPartial.cshtml**

Since the control elements related to the scripts are included in the partial view **CustomerEditPartial.cshtml**, it will be helpful to look at the partial view when examining the functionality of the scripts.

#### CustomerEditPartial.cshtml

~~~html
@model Blip.Entities.Customers.ViewModels.CustomerEditViewModel

@{
    Layout = null;
}

@using (Html.BeginForm("EditCustomerPartial", "Customer", FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <div class="form-horizontal">
        <hr />
        <h4>Edit customer details</h4>
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        <div class="form-group">
            @Html.LabelFor(model => model.CustomerID, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.CustomerID, new { htmlAttributes = new { @class = "form-control", @readonly = "readonly" } })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.CustomerName, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.CustomerName, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.CustomerName, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.SelectedCountryIso3, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.DropDownListFor(model => model.SelectedCountryIso3, new SelectList(Model.Countries, "Value", "Text"), htmlAttributes: new { @class = "form-control", @id="Country" })
                @Html.ValidationMessageFor(model => model.SelectedCountryIso3, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.SelectedRegionCode, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.DropDownListFor(model => model.SelectedRegionCode, new SelectList(Model.Regions, "Value", "Text"), htmlAttributes: new { @class = "form-control", @id="Region" })
                @Html.ValidationMessageFor(model => model.SelectedRegionCode, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Save" class="btn btn-primary" />
            </div>
        </div>
    </div>
}
~~~

Note the `@Html.DropDownListFor` helper methods for the `@id="Country"` and `@id="Region"` elements.

Also note the following Razor code:

~~~html
@{
    Layout = null;
}
~~~

Because **CustomerEditPartial.cshtml** is the first partial view loaded in the **Edit.cshtml** parent view and because **Edit.cshtml** does not have an `@Model` directive, it is necessary to tell the Razor engine not to use _Layout.cshtml for the partial view. Without this directive the Razor engine gets confused and renders elements from the layout file, such as the footer.

### Scripts for **CustomerEditPartial.cshtml**

The use case for the form for editing the Country and State/Region associated with the customer is somewhat different from that of adding a new postal address, so the script functionality is also somewhat different. When the **Customer/Edit** page is used, the customer already exists, so there is an existing record including values for the customer's Country and State/Region. Accordingly, the script needs to populate the list of values for the **State/Region** field with values associated with the existing value for **Country** and set the value of the **Region** control accordingly, as follows in the first of the JavaScript functions in **Customer/Edit.cshtml**, shown below.

Note that the implementation-specific JavaScript uses the jQuery library ("`$`").

Ellipsis ("`...`") indicates places where irrelevant code has been redacted for brevity.

~~~html
<script type="text/javascript">
    $(document).ready(function () {
        var selectedCountry = $("#Country").val();
        var selectedRegion = $("#Region").val();
        var regionsSelect = $('#Region');
        AddRegions(selectedCountry, regionsSelect);
        if (selectedRegion != null && selectedRegion != '') {
            regionsSelect.val = selectedRegion;
        };
    });
...
</script>
~~~

This function gets the value of the `Country` and `Region` elements provided by the view model **CustomerEditViewModel.cs**, calls the `AddRegions` function to add the appropriate values to the `Region` element, and resets the value of the `Region` element.

The `AddRegions` function calls the `CustomerController.GetRegions` method to get a list of the regions for the country identified in the value for the `Region` element. It then appends these key-value pairs to the list of values for the `Region` element, adding a blank record before doing so to present the dropdown in an unselected state. As noted above, the `(docment).ready` function for **Edit.cshtml** resets the selected value for `Region` once the list is loaded.

~~~html
<script type="text/javascript">
...
function AddRegions(selectedCountry, regionsSelect) {
    $.getJSON('@Url.Action("GetRegions")', { iso3: selectedCountry }, function (regions) {
        if (regions != null && !jQuery.isEmptyObject(regions)) {
            regionsSelect.append($('<option/>', {
                value: null,
                text: ""
            }));
            $.each(regions, function (index, region) {
                regionsSelect.append($('<option/>', {
                    value: region.Value,
                    text: region.Text
                }));
            });
        };
    });
</script>
~~~

For more information on the `GetRegions` controller action, see the [BlipAjax](https://github.com/ajsaulsberry/BlipAjax) example solution on GitHub.

#### A note about script binding for partial views

As noted above, the convention for JavaScript associated with Razor partial views is to include them in the parent view **.cshtml**. JavaScript functions can be bound to elements on the partial view the partial view is rendered at the same time as the parent view. This happens when loading the partial view with an `@Html.Action` helper method, as in this section from **Edit.cshtml**:

~~~html
...
<div id="EditCustomerPartial">
    @Html.Action("EditCustomerPartial", new { id = Url.RequestContext.RouteData.Values["id"] })
</div>
...
~~~

The partial views **CustomerEditPartial.cshtml** and **AddressTypePartial.cshtml** enable the "componentization" of the **Customer/Edit.cshtml** view. Both partial views are rendered at the same time as the parent view, and thus the `Country` and `Region` elements are available for binding when the `document.ready` event occurs.

### Scripts for **CreatePostalAddressPartial.cshtml**

Rendering of the partial view **CreatePostalAddressPartial.cshtml** is initiated by the `HttpPost` event of the Ajax form created by the **AddressTypePartial.cshtml** partial view, defined as follows:

#### AddressTypePartial.cshtml

~~~csharp
@model Blip.Entities.Customers.ViewModels.AddressTypeViewModel

@using (Ajax.BeginForm("AddressTypePartial", new AjaxOptions { HttpMethod = "POST", UpdateTargetId = "CreateAddress", InsertionMode = InsertionMode.Replace }))
{
    @Html.AntiForgeryToken()
...
}
~~~

Despite the `Html.BeginForm` helper method form, the controller action `CustomerController.AddressTypePartial(AddressTypeViewModel model)` kicks off the rendering of the Ajax partial view rather than an `HttpGet` action for the **CreatePostalAddressPartial.cshtml** partial view. The partial view replaces the empty `<div id="CreateAddress"></div>` element of **Edit.cshtml**. (For a more detailed explanation of using Ajax partial views, see the companion guide )

Because the postal address partial view is not rendered when the parent view and the preceding partial views are rendered, JavaScript loaded with the parent view **Edit.cshtml** cannot be bound to elements on the postal address partial view. 
> Accordingly, the scripts associated with the Ajax partial view must be included with the partial view, rather than the parent view. This is a key exception to the coding convention, necessitated by script binding.

Note that some of the form elements have been redacted for brevity, as shown by ellipsis ("`...`").

#### CreatePostalAddressPartial.cshtml

~~~html
@model Blip.Entities.Customers.ViewModels.PostalAddressEditViewModel

<script type="text/javascript">
    $("#AddressCountry").change(function () {
        var selectedCountry = $("#AddressCountry").val();
        var regionsSelect = $("#AddressRegion");
        regionsSelect.empty();
        if (selectedCountry != null && selectedCountry != '') {
            AddRegions(selectedCountry, regionsSelect);
        }
    });
</script>

@using (Html.BeginForm("CreatePostalAddressPartial", "Customer", FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <div class="form-horizontal">
        <hr />
        <h4>Add a new postal address</h4>
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        @Html.HiddenFor(model => model.CustomerID)
        <div class="form-group">
            @Html.LabelFor(model => model.PostalAddressID, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.PostalAddressID, new { htmlAttributes = new { @class = "form-control", @readonly = "readonly" } })
            </div>
        </div>
 ...
        <div class="form-group">
            @Html.LabelFor(model => Model.SelectedCountryIso3, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-5">
                @Html.DropDownListFor(model => Model.SelectedCountryIso3, new SelectList(Model.Countries, "Value", "Text"), htmlAttributes: new { @class = "form-control", @id = "AddressCountry" })
                @Html.ValidationMessageFor(model => Model.SelectedCountryIso3, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="form-group">
            @Html.LabelFor(model => Model.SelectedRegionCode, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-5">
                @Html.DropDownListFor(model => Model.SelectedRegionCode, new SelectList(Model.Regions, "Value", "Text"), htmlAttributes: new { @class = "form-control", @id = "AddressRegion" })
                @Html.ValidationMessageFor(model => Model.SelectedRegionCode, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Create" class="btn btn-primary" />
            </div>
        </div>
    </div>
}
~~~

Note the following features of the partial view:

* The `<script>` element is not enclosed in a scripts section.

* The names of the elements for the country and region fields, `AddressCountry` and `AddressRegion` are distinct from the names of the similar elements on the **CustomerEditPartial.cshtml** partial view. This is essential for proper binding: be sure elements have distinct `id` and `name` attributes, as appropriate.

* The event that triggers population of the values for the `AddressRegion` dropdown is the `.change` event for the `AddressCountry` element.

* Although the script is loaded with the partial view, it can reference the libraries and scripts on the **Edit.cshtml** parent view, including the jQuery library ("`$`") and the function `AddRegions`.

The final point is a powerful one for using custom scripts with Ajax partial views. The partial view rendered with unobtrusive Ajax needs to include just enough script code to take care of element binding and calls to libraries and functions in the parent, helping to prevent duplication of code.

Also note that the unobtrusive client-side validation loaded with the `jqueryval` bundle in the parent view will work on the form elements in the partial view loaded with Ajax.

## Conclusion

When structured properly, JavaScript code can extend the power of JavaScript libraries and custom code to Razor partial views rendered with the unobtrusive Ajax library. The key steps are:

* load jQuery in _Layout.cshtml

* load jquery-unobtrusive-ajax.js, jquery.validate.js, and jquery.validate.unobtrusive.js in the `Scripts` section of appropriate pages

* include scripts for partial views rendered with the parent view in the parent view

* include scripts for partial views rendered with Ajax in the partial view

* include common functions in the `Scripts` section of the parent view.

With these techniques it's possible to create powerful, interactive, data-driven web pages with minimal code and without having to strap on the development overhead of implementing a client-side framework.

## More information

If you want to dive deeper into the topics discussed in this guide or experiment with the code shown above, the following is a  selected list of resources.

### Example project

The complete Visual Studio solution described in this guide is available on GitHub:

[BlipAjax](https://github.com/ajsaulsberry/BlipAjax)

The example project is provided as is, without any warranty expressed or implied. Neither PluralSight nor the author is responsible for any errors or omissions.

### Related PluralSight courses

[ASP.NET MVC Advanced Topics](https://app.pluralsight.com/library/courses/aspdotnet-mvc-advanced-topics/table-of-contents) by Scott Allen, 22 July 2009. The first module of this course, *Ajax with ASP.NET MVC* provides a nice narrated overview of the related technology.

### Other resources

[learn.jquery.com](http://learn.jquery.com/) &mdash; This is the official site for jQuery documentation, provided by the jQuery Foundation.

[Microsoft: Bundling and Minification](https://docs.microsoft.com/en-us/aspnet/mvc/overview/performance/bundling-and-minification), by Rick Anderson, 23 August 2012 &mdash; This is the essential guide to implementing bundling and minification in ASP.NET MVC. If you are writing for mobile devices &ndash; and you should be &ndash; Google will down-rate your page performance unless you use bundling and minification, hurting your performance in search.

[Microsoft: System.Web.Mvc namespace, `AjaxHelper<TModel>` Class](https://msdn.microsoft.com/en-us/library/dd470355) and [Microsoft: System.Web.Mvc.Ajax namespace](https://msdn.microsoft.com/en-US/library/system.web.mvc.ajax) &mdash; These are the canonical documentation references for the AjaxHelper class, AjaxExtensions (including the BeginForm method), and AjaxOptions classes. Note that if you go looking for information on docs.microsoft.com it will boot you over to these URL's. Although this page refers to Visual Studio 2013 it is, in fact, the most recent documentation.

[Wikipedia: Ajax (programming)](https://en.wikipedia.org/wiki/Ajax_(programming)) &mdash; A nice summary of the Ajax architecture, history, and implementation. Good background for beginners looking for a fundamental grounding in the technology.

---

Thank you for reading! Feel free to drop any comments or questions in the discussion section below and favorite this article if you found it interesting or informative.

Written by A. J. Saulsberry, 20 October 2017