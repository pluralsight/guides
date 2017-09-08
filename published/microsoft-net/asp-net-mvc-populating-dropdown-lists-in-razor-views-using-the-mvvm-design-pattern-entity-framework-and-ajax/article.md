## Introduction

This guide presents a couple common ways to populate dropdown lists in ASP.NET MVC Razor views, with an emphasis on producing functional HTML forms with a minimum amount of code. It is intended to help developers who are working to improve their proficiency with some key technologies.

Also shown is how the contents of one dropdown list can be updated dynamically based on the selection made in another dropdown list.

### Skill levels

It will be helpful to have an understanding of the following topics, but extensive working knowledge isn't required.

| **Technology** | **Skill Level** |
| :--- | :--- |
| Ajax | Introductory
| ASP.NET | Beginner
| C# | Beginner
| Entity Framework | Beginner
| JavaScript | Beginner
| jQuery | Introductory
| MVC | Beginner

### Scope

This guide focuses on ASP.NET MVC following the model-view view-model (MVVM) design pattern. The solution includes a small amount of JavaScript utilizing the [jQuery](http://jquery.com/) library, in addition to the required C# code. Entity Framework is used to handle the interface between the SQL Server database and the data entities in the model.

### Structure

1. We'll start with the essential code for implementing a dropdown list populated when the view model is created.

1. Building on that functionality, we'll look at how to populate the State/Province/Region dropdown based on the country selected.

1. Next we'll look at the entities underlying the case study and how the pieces fit together.

1. Finally, we'll touch on some considerations for building production solutions.

### Case study

A complete Visual Studio solution with all the code shown in this guide is available from [GitHub](https://github.com/ajsaulsberry/BlipDrop).

### Conventions

In each of the code segments below we'll see the file name from the associated sample project, like this:

#### BlipDrop.csproj

We'll also see the `using` statements so you can better understand how the components reference each other.

Ellipsis (`...`) in the code segments indicate that in the complete file there is code above or below the segment that is unrelated to the topic at hand.

## Creating a dropdown list from the view's bound view model

Implementing a dropdown list with data supplied in a view model involves four components:

1. View model,
1. Razor view,
1. Controller action, and
1. Repository method.

We'll look at the essential code for each of these. If you need a better understanding of how these pieces of code fit into each component, the following section will provide a higher level picture of all the components.

### View model

Using a view model as the model for the form containing the dropdown elements separates presentation from structure. Keeping things normalized happens on the back end.

In our example, we're creating a new Customer record. Customers have Country and Region (state or province) properties.

#### CustomerDisplayViewModel.cs

~~~csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Web.Mvc;

namespace BlipDrop.ViewModels
{
    public class CustomerEditViewModel
    {
        [Display(Name = "Customer Number")]
        public string CustomerID { get; set; }

        [Required]
        [Display(Name = "Customer Name")]
        [StringLength(75)]
        public string CustomerName { get; set; }

        [Required]
        [Display(Name = "Country")]
        public string SelectedCountryIso3 { get; set; }
        public IEnumerable<SelectListItem> Countries { get; set; }

        [Required]
        [Display(Name = "State / Region")]
        public string SelectedRegionCode { get; set; }
        public IEnumerable<SelectListItem> Regions { get; set; }
    }
}
~~~

Notice that for each property for which we're going to provide dropdown lists we have two fields in the view model:

1. one for the **list**
1. one for the **selected item**

The **list** is made up of a collection of `SelectListItem` type.

The field storing the **selected item** holds the unique key for each entity. The unique key of the selected value will be the same as one of the elements of the `SelectListItem`.

### Razor view

These are the form elements for the Country field on the Razor view showing the implementation of the `DropDownListFor` HtmlHelper.

#### Create.cshtml, Country snippet

~~~html
...
<div class="form-group">
    @Html.LabelFor(x => Model.SelectedCountryIso3, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-5">
        @Html.DropDownListFor(x => Model.SelectedCountryIso3, new SelectList(Model.Countries, "Value", "Text"), htmlAttributes: new { @class = "form-control", id = "Country"})
        @Html.ValidationMessageFor(x => x.SelectedCountryIso3, "", new { @class = "text-danger" })
    </div>
</div>
...
~~~

Compare the `@Html.DropDownListFor` statement to the view model.

`x => Model.SelectedCountryIso3` identifies the field in the view model where the selected value will be stored. We're storing the unique identifier for the Country.

`new SelectList(Model.Countries, "Value", "Text")` identifies the source of the list to use in populating the dropdown list, `IEnumerable<SelectListItem> Countries` from the data model. Note that it does this by creating a new `SelectList` with fields `Value` and `Text`, which correspond to the same fields in the Countries property of the data model.

**You cannot** use a field of type `SelectList` in your view model. The Razor engine needs to get the data as a collection of `SelectListItems` to build the HTML element properly. This is documented as opaquely as possible in the [framework documentation](https://msdn.microsoft.com/en-us/library/system.web.mvc.html.selectextensions.dropdownlistfor(v=vs.118).aspx) on MSDN.

Note that the HTML element on the rendered page will have an `id` attribute value of `Country`. JavaScript code we'll look at later will use the `id` attributes of each of the dropdown elements.

The complete view looks like this when rendered by the browser:

#### /Customer/Create

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/58514c4d-1ddc-4368-9b37-c5f35bc2d6b6.png)

### Controller actions

Your controller can be simple: your controller just has to call the appropriate repository to get the view model, then you pass the view model to the view:

#### CustomerController.cs, Index action

~~~csharp
using System.Collections.Generic;
using System.Web.Mvc;
using BlipDrop.Data;
using BlipDrop.ViewModels;

namespace BlipDrop.Controllers
{
    public class CustomerController : Controller
    {
        // GET: Customer
        public ActionResult Index()
        {
            var repo = new CustomersRepository();
            var customerList = repo.GetCustomers();
            return View(customerList);
        }
    ...
    }
}
~~~

Note in the `using` statements that the controller references the data context and the view models, but doesn't need to access the entities which map to database objects.

### Repository methods

In our simple example the customer repository creates an instance of the `CustomerEditViewModel`
class and assigns a new GUID to the `CustomerID` field.

It also calls the countries and regions repositories to gets the list of countries and regions so the view model with have the data for the dropdown lists on the view.

#### CustomerRepository.cs

~~~csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using BlipDrop.Models;
using BlipDrop.ViewModels;

namespace BlipDrop.Data
{
    public class CustomersRepository
    {
    ...
        public CustomerEditViewModel CreateCustomer()
        {
            var cRepo = new CountriesRepository();
            var rRepo = new RegionsRepository();
            var customer = new CustomerEditViewModel()
            {
                CustomerID = Guid.NewGuid().ToString(),
                Countries = cRepo.GetCountries(),
                Regions = rRepo.GetRegions()
            };
            return customer;
        }
    ...
    }
}
~~~

Note that in this scenario we don't need to use an instance of the data context, `ApplicationDbContext`,  to create an instance of the `CustomeEditViewModel` because only the `Countries` and `Regions` fields require calls to the database to get values, and those calls are handled by the respective repositories for those objects.

#### CountriesRepository.cs

~~~csharp
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;

namespace BlipDrop.Data
{
    public class CountriesRepository
    {
        public IEnumerable<SelectListItem> GetCountries()
        {
            using (var context = new ApplicationDbContext())
            {
                List<SelectListItem> countries = context.Countries.AsNoTracking()
                    .OrderBy(n => n.CountryNameEnglish)
                        .Select(n =>
                        new SelectListItem
                        {
                            Value = n.Iso3.ToString(),
                            Text = n.CountryNameEnglish
                        }).ToList();
                var countrytip = new SelectListItem()
                {
                    Value = null,
                    Text = "--- select country ---"
                };
                countries.Insert(0, countrytip);
                return new SelectList(countries, "Value", "Text");
            }
        }
    }
}
~~~

Note that the return type for the CountryRepository is the same type as the Countries list in the view model, `IEnumerable<SelectListItem>`. This is the same collection as used by the `@Html.DropDownListFor` statement in the Razor view. The `Value` element of each `SelectListItem` is the Country's `Iso3` code converted from a GUID to text.

`RegionsRepository.GetRegions()` returns an empty `SelectListItem` collection. This is so that the view model doesn't pass a null object reference to the Region dropdown list.

#### RegionsRepository.cs, Part 1

~~~csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;

namespace BlipDrop.Data
{
    public class RegionsRepository
    {
        public IEnumerable<SelectListItem> GetRegions()
        {
            List<SelectListItem> regions = new List<SelectListItem>()
            {
                new SelectListItem
                {
                    Value = null,
                    Text = " "
                }
            };
            return regions;
        }
    }
}
~~~

Initializing a dependent dropdown in this way also gives you an opportunity to display an informational message (e.g., "select a country first") before populating the list values.

## Populating a dropdown based on the selection in another dropdown

This techique adds two more pieces of code to the ones above:

* an additional HttpGet controller action and
* JavaScript on the view.

### Razor view, regions section

The Create view for the includes a dropdown for Regions that works just like the one for Countries:

#### Create.cshtml, Region snippet

~~~csharp
...
<div class="form-group">
    @Html.LabelFor(x => Model.SelectedRegionCode, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-5">
        @Html.DropDownListFor(x => Model.SelectedRegionCode, new SelectList(Model.Regions, "Value", "Text"), htmlAttributes: new { @class = "form-control", @id = "Region" })
        @Html.ValidationMessageFor(x => x.SelectedRegionCode, "", new { @class = "text-danger" })
    </div>
</div>
...
~~~

Remember that we've initialized the Regions field of the view model with an empty list. We'll need to go and get some values for the list when the user selects a country.

### Razor view, JavaScript code

Populating the list values for a dropdown field on an HTML for requires an event on the client side. This technique uses the jQuery JavaScript library to bind an event handler to the Change event of the Country dropdown. Whenever the user selects a value in the Country dropdown the event fires and the JavaScript code executes.

#### Create.cshtml, Scripts snippet

~~~javascript
@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
    <script type="text/javascript">
            $('#Country').change(function () {
                var selectedCountry = $("#Country").val();
                var regionsSelect = $('#Region');
                regionsSelect.empty();
                if (selectedCountry != null && selectedCountry != '') {
                    $.getJSON('@Url.Action("GetRegions")', { iso3: selectedCountry }, function (regions) {
                        if (regions != null && !jQuery.isEmptyObject(regions))
                        {
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
            });
    </script>
}
~~~

This looks like a lot if you are relatively new to JavaScript, but it's easy to unpack:

1. When the `change` event of the `Country` HTML element (the dropdown list) fires:
    1. get the value of the element and store it in the `selectedCountry` variable.
    1. Clear out anything that was previously in the `Region` dropdown list.
1. If the `Country` field contains a value (and it might not because the first element of the list of countries is a blank item):
    1. Call the `GetRegions(string Iso3)` controller action, passing it the value selected in `Country` and storing the results in the object variable `regions`.
    1. If `regions` is not null or empty (which it can be if there are no regions associated with a country):
        1. Create a blank element as the first element of the list.
        1. Iterate through the collection in `regions` and add each element to the dropdown's list of values.

The jQuery functions begin with `$`. For more information on the functions used see:

[`change`](http://learn.jquery.com/events/event-basics/)
[`getJSON`](http://learn.jquery.com/ajax/jquery-ajax-methods/)
[`each`](http://learn.jquery.com/using-jquery-core/iterating/)

### Controller action

The JavaScript on the page (Create.cshtml) calls a controller action to get the data from the server asynchronously. The controller action's job is to call the repository to get the data, then convert it from an IEnumerable of SelectListItems to JSON format and pass it back to the page.

#### CustomerController.cs, GetRegions action

~~~csharp
using System.Collections.Generic;
using System.Web.Mvc;
using BlipDrop.Data;
using BlipDrop.ViewModels;

namespace BlipDrop.Controllers
{
    public class CustomerController : Controller
    {
    ...
        [HttpGet]
        public ActionResult GetRegions(string iso3)
        {
            if (!string.IsNullOrWhiteSpace(iso3) && iso3.Length == 3)
            {
                var repo = new RegionsRepository();

                IEnumerable<SelectListItem> regions = repo.GetRegions(iso3);
                return Json(regions, JsonRequestBehavior.AllowGet);
            }
            return null;
        }
    ...
}
~~~

Note that while it works perfectly well to pass an IEnumberable to a view using ASP.NET model binding, that doesn't work for Ajax calls. Model binding takes care of converting the IEnumerable to HTML.

### Repository method

RegionsRepository.cs contains two signatures for `GetRegion`:

`GetRegions()` which returns an empty list of regions and
`GetRegions(string iso3)` which returns the list of regions associated with the country code passed in the parameter.

We looked at the first signature when we looked at the [Regions repository](#RegionsRepository.cs,-Part-1) for the first time above.

This method works much like the `GetCountries` method, except that the `ios3` parameter is used to select a subset of the records from the Regions table of the database.

#### RegionsRepository.cs, part 2

~~~csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;

namespace BlipDrop.Data
{
    public class RegionsRepository
    {
    ...
        public IEnumerable<SelectListItem> GetRegions(string iso3)
        {
            if (!String.IsNullOrWhiteSpace(iso3))
            {
                using (var context = new ApplicationDbContext())
                {
                    IEnumerable<SelectListItem> regions = context.Regions.AsNoTracking()
                        .OrderBy(n => n.RegionNameEnglish)
                        .Where(n => n.Iso3 == iso3)
                        .Select(n =>
                           new SelectListItem
                           {
                               Value = n.RegionCode,
                               Text = n.RegionNameEnglish
                           }).ToList();
                    return new SelectList(regions, "Value", "Text");
                }
            }
            return null;
        }
    }
}
~~~

Note that we're setting the order of the elements to be the region name, rather than `RegionCode`. This is so the user sees the list of regions, like US states, in a predicable alphabetical order, rather than an order dictated by the index value.

## Case study entities

The case study is a (very) simple list of customers. Each is assigned a GUID as a unique identifier. Their name, country, and region define each Customer object.

Countries can have zero or more regions.

### Database schema
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/10a455aa-45bf-442f-a6f5-83dee7db5df9.png)

*Relational database schema diagram for multiple dropdowns example project*

### Data entities

The classes for the data entities form the Object side of the Object Relational Mapper (ORM) that is Entity Framework. Using code-first entity migrations, as is done in this project, enables the database tables and their properties, including their relationships, to be defined in code. By defining the database in this fashion, it can be kept in synchronization with the entities for which it provides a persistent data store.

#### Customer.cs

~~~csharp

using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BlipDrop.Models
{
    public class Customer
    {
        [Key]
        [Column(Order = 1)]
        public Guid CustomerID { get; set; }

        [Required]
        [MaxLength(128)]
        public string CustomerName { get; set; }

        [Required]
        [MaxLength(3)]
        public string CountryIso3 { get; set; }

        [MaxLength(3)]
        public string RegionCode { get; set; }

        public virtual Country Country { get; set; }

        public virtual Region Region { get; set; }
    }
}
~~~

In the customer class we can identify the required fields and set the maximum length of string fields (as well as other attributes). Entity Framework uses these attributes when creating and modifying the database.

The maximum length specified in the entity class defines the maximum size of the database field. In the [view model](#CustomerDisplayViewModel.cs) the `StringLength` attribute is used to identify the maximum size of the field input.

> In the case study we've done a couple seemingly inconsistent things to demonstrate how Entity Framework and the model-view view-model (MVVM) design pattern work together:
> * In the view model the CustomerName field is `[StringLength(75)]` but the database field size is `[MaxLength(128)]`, demonstrating that input requirements can be different than database requirements. (But making the input size larger than the database size can cause problems.)
> * In the view model the RegionCode is `Required`, but it's an optional field in the database. This is to demonstrate the error handling HtmlHelpers.

Database relationships are defined with *navigation properties* like the one for the Customer-Country relationship:

~~~ csharp
public virtual Country Country { get; set; }
~~~

This defines the foreign key relationship the Country table.

See the [More information](#More-information) section for learning resources if you need a better understanding of Entity Framework.

#### Country.cs

~~~csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace BlipDrop.Models
{
    public class Country
    {
        [Key]
        [MaxLength(3)]
        public string Iso3 { get; set; }

        [Required]
        [MaxLength(50)]
        public string CountryNameEnglish { get; set; }

        public virtual IEnumerable<Region> Regions { get; set; }
    }
}
~~~

This class has many of the same features as the Customer class, but in this case the navigation property identifies a one-to-many relationship between Countries and Regions.

#### Region.cs

~~~csharp
using System.ComponentModel.DataAnnotations;

namespace BlipDrop.Models
{
    public class Region
    {
        [Key]
        [MaxLength(3)]
        public string RegionCode { get; set; }

        [Required]
        [MaxLength(3)]
        public string Iso3 { get; set; }

        [Required]
        public string RegionNameEnglish { get; set; }

        public virtual Country Country { get; set; }
    }
}
~~~

In the sample project each region has a unique RegionCode. In real life, the unique identifier for a region is more likely to be a compound code consisting of the country identifier and the region code. For the sake of simplicity, we've simplified the structure.

## Production considerations

Turning an example project like this into production application usually means adding code to address a number of requirements:

* localization (language and conventions),
* error messages for the user interface,
* error handling and logging,
* unit tests, and
* deployment.

There are also some considerations with respect to the use of the techniques shown above, particularly relating to performance.

When web pages make server calls to get data there is some latency involved in the round trip to the server, and this delay can increase when the server is under load. Two things are often done to ameliorate this condition:

1. implement the controller actions and repository methods asynchronously and
1. display a progress indicator while the client is waiting for a response from the server.

Depending on the data in question, it may be better to transfer all the necessary data to the client and perform the responsive queries locally. For example, in addition to sending all the country records in the view model, all the region records would be sent as well. The regions associated with the country selected by the user would be populated in the Region dropdown list with JavaScript code running locally.

## More information

If you want to dive deeper into the topics discussed in this guide, the following is a  selected list of resources.

### Related PluralSight courses

[PluralSight](https://www.pluralsight.com) offers a number of courses on the topics mentioned in this guide. The following are some suggestions organized by technology:

| **Technology** | **Course(s)** |
| :--- | :--- |
| Ajax | [ASP.NET Ajax JavaScript and jQuery](https://app.pluralsight.com/library/courses/aspdotnet-ajax-jscript/table-of-contents)
| ASP.NET | [Building Applications with ASP.NET MVC 4](https://app.pluralsight.com/library/courses/mvc4-building/table-of-contents)
| C# | [C# Fundamentals with Visual Studio 2015](https://app.pluralsight.com/library/courses/c-sharp-fundamentals-with-visual-studio-2015/table-of-contents)
| Entity Framework | [Getting Started with Entity Framework 6](https://app.pluralsight.com/library/courses/entity-framework-6-getting-started/table-of-contents)]
| JavaScript | [JavaScript Fundamentals for ES6](https://app.pluralsight.com/library/courses/javascript-fundamentals-es6/table-of-contents)
| jQuery | [ASP.NET Ajax JavaScript and jQuery](https://app.pluralsight.com/library/courses/aspdotnet-ajax-jscript/table-of-contents)
| MVC | [ASP.NET MVC 5 Fundamentals](https://app.pluralsight.com/library/courses/aspdotnet-mvc5-fundamentals/table-of-contents)

### Case study code on GitHub

The complete Visual Studio solution for the code shown above is available on GitHub at:

[https://github.com/ajsaulsberry/BlipDrop](https://github.com/ajsaulsberry/BlipDrop)

You can fork the project, run the code, and experiment on your own.

Note that the sample project is not intended to be a real life case study or production code; it exists to illustrate some techniques.

### Other resources

Here are some valuable links for improving your knowledge of topics covered in this guide:

[The ASP.NET website](https://www.asp.net/mvc)
[docs.microsoft.com C# documentation](https://docs.microsoft.com/en-us/dotnet/csharp/csharp)

[docs.microsoft.com Entity Framework documentation](https://docs.microsoft.com/en-us/ef/)

[JavaScript.com/learn](https://www.javascript.com/learn/javascript/strings)

[jQuery Learning Center](http://learn.jquery.com/)  
This is a spiffy site from [www.codeschool.com](https://www.codeschool.com), a PluralSight company

*Written by A. J. Saulsberry, 18 August 2017.*