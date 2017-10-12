#### *Quick-Hit answer*

Developers usually stumble with default binding when handling the `HttpPost` action. For example; if you have view, view model, controller actions, and data all wired-up and it looks like your form should be working, but your model is empty or partially empty when it hits the controller after you press "Save", you probably need to make some adjustments to the view to get the Razor engine to compose the HTML properly. Getting the code for the client side right is essential, and it fails silently when it's wrong.

If you just need help with that, jump to the section on [**Saving data**](#saving-data).

If you feel you would benefit from a more complete case study, read on.

## Introduction

Many developers know they can create forms on web pages with a minimum of code using ASP.NET model binding. Visual Studio's default MVC view templates will even create standard list, create, edit, and delete views without any additional programming.

But the power of default model binding extends beyond the flat data model of a simple input form or list of records. Using a few straightforward coding techniques, developers can use ASP.NET to create forms and collect data for hierarchical entity relationships. In many applications this can make the difference between leveraging the rapid development capabilities of ASP.NET MVC and strapping on the additional infrastructure and complexity of a client-side framework like Angular or React.

This guide will present an example of using ASP.NET MVC model binding to present and collect hierarchical form data in a hierarchical structure.

### Skill levels

It will be helpful to have an understanding of these topics at the indicated skill level:

| **Technology** | **Skill Level** |
| :--- | :--- |
| ASP.NET | Intermediate
| C# | Intermediate
| Entity Framework | Beginner
| MVC | Intermediate
| MVVM | Beginner

### Scope

This guide will present an example of using ASP.NET MVC model binding to present and collect hierarchical form data in a hierarchical structure. The focus

The example project and the code presented in this guide are based on the .NET Framework. Implementation details for .NET Core and .NET Standard will be covered in a different guide.

### Structure

1. We'll begin with an overview of the case study entities and the principal views of the example solution used to prepare this guide.

1. Then we'll see how to create a view model incorporating member fields of various primitive types and incorporating a field that is a collection of an object type.

1. Next we'll look at the code required to present information to the end user.

1. We'll conclude with a close look at how to ensure that Razor code creates the correct HTML and we'll take a look how to use HtmlHelpers and CSS to apply formatting to the form fields created by the view.

### Case study

The code and screenshots shown in this guide match the code and view layouts an example Visual Studio project. The solution can be forked or downloaded from a GitHub repository:

[github.com/ajsaulsberry/BlipBinding](https://github.com/ajsaulsberry/BlipBinding)

The sample solution implements the following:

* a multi-project solution with separate projects for
  * web application,
  * entities,
  * data layer (context and repositories);
* the Model-View ViewModel (MVVM) design pattern;
* the repository design pattern; and
* the Entity Framework ORM with code-first development.

Using the example solution you can follow along with each section below and experiment on your own.

### Prerequisites

You should have:

* a working knowledge of ASP.NET MVC
* an understanding of the Model View ViewModel (MVVM) design pattern
* Visual Studio ready to go

## BlipBinding case study solution

The case study implemented by the [BlipBinding](https://github.com/ajsaulsberry/BlipBinding) solution is a simple application for maintaining information about customers, their orders, and the items in their orders. The permanent data store for the application is a SQL Server database. The tables and their relationships are shown below:


![BlipBinding entity relationship diagram](https://raw.githubusercontent.com/pluralsight/guides/master/images/485ca4b0-cbd4-41a5-9cc4-bc181ff3598e.png)

*BlipBinding database entity-relationship diagram*

### Entities and relationships

Note that the many-to-many relationship between **Orders** and **Items** is implemented through the use of a *merge table with payload*: in addition to maintaining the relationship between **Orders** and **Items**, the **OrderItems** table also contains information about the items included in an order, the price at which they were sold and the quantity which were sold.

Obviously, this isn't a complete order processing system; it's just meant to provide an example of hierarchical relationships in a familiar form.

The database is created and maintained using Entity Framework code-first design. Each table and it's relationship to other tables is defined by a class in the **Blip.Entities** project of the **BlipBinding** solution. Let's look at the `Customer` and `Order` entities:

#### Blip.Entities\Customers\Customer.cs

~~~csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Blip.Entities.Geographies;
using Blip.Entities.Orders;

namespace Blip.Entities.Customers
{
    public class Customer
    {
        public Customer()
        {
            Orders = new HashSet<Order>();
        }

        [Key]
        [Column(Order = 0)]
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
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

        public virtual ICollection<Order> Orders { get; set; }
    }
}
~~~

Note the following characteristics:

* ComponentModel DataAnnotations are used to identify the key field for the database and to specify the size and other options.

* The one-to-many relationship between **Customers** and **Orders** is created by the `virtual` member field comprised of a collection of `Order` entities.

* The required relationship between **Customers** and **Countries** in the database is reflected in the field to hold the value `CountryIso3` and the field to identify the relationship, `Country`, which is of type `Country`.

#### Order.cs

The `Order` entity is defined in a similar way:

~~~csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Blip.Entities.Customers;
using Blip.Entities.Items;

namespace Blip.Entities.Orders
{
    public class Order
    {
        public Order()
        {
            Items = new HashSet<Item>();
        }

        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public Guid OrderID { get; set; }

        [Required]
        public Guid CustomerID { get; set; }

        [Required]
        public DateTime OrderDate { get; set; }

        [Required]
        [MaxLength(128)]
        public string Description { get; set; }

        public virtual ICollection<Item> Items { get; set; }

        public virtual Customer Customer { get; set; }
    }
}
~~~

Note that:

* The `Order` can only belong to one `Customer`, as reflected in the field for a single `CustomerID` and the navigation property `Customer`.

* The one-to-many relationship between **Orders** and **Items** is implemented through the virtual property for the collection of `Item` entities.

The data context and Entity Framework code-first migrations for the database are located in the **Blip.Data** project, along with the repository methods.

## Presenting data

The **Blip.Web** project in the **BlipBinding** solution is based on the standard .NET Framework MVC template, so the layout of the views is based on the default Bootstrap CSS styling and the `_layout.cshtml` included with the template.

For the purposes of this guide, there are two notable views in the case study, one to display a list of customers and another to display a list of orders for each customer.

### Customer/Index view

The view for the list of customers is a simple table displaying some basic information about the customer and an `Html.ActionLink` helper method to navigate to the list of orders for the customer:


![BlipBinding Customer/Index view](https://raw.githubusercontent.com/pluralsight/guides/master/images/01390b7a-fcda-4e6a-ae33-0743cb771bd1.png)

*BlipBinding solution, Blip.Web project, Customer/Index view*

Using the `Seed` method of Entity Framework code-first migrations, we have populated the database with a few customers, orders, and items. If you run the example solution the application will create the database and add the same records. (The GUID's created by your computer for the key fields will be different than those shown.)

#### Blip.Web\Views\Customer\Index.cshtml

In the code for the view above, note that the list of customers is created with a `foreach` loop that iterates through the collection of `CustomerDisplayViewModel` entities. This is a standard way of presenting a list with a variable number or records:

~~~csharp
@foreach(var item in Model)
{
    <tr>
        <td>
            @Html.DisplayFor(modelItem => item.CustomerID)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.CustomerName)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.CountryName)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.RegionName)
        </td>
        <td>
            @Html.ActionLink("Orders", "Index", "Order",  new { customerid = item.CustomerID }, null)
        </td>
    </tr>
}
~~~

In the list of customers, the view model has a flat structure, it's just an enumerable list of objects that contain the customer information. Here's the `@model` directive from the beginning of `Index.cshtml`:

~~~csharp
@model IEnumerable<Blip.Entities.Customers.ViewModels.CustomerDisplayViewModel>
~~~

Now let's take a closer look at that view model.

#### Blip.Entities\Customers.ViewModels\CustomerDisplayViewModel.cs

~~~csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace Blip.Entities.Customers.ViewModels
{
    public class CustomerDisplayViewModel
    {
        [Display(Name = "Customer Number")]
        public Guid CustomerID { get; set; }

        [Display(Name = "Customer Name")]
        public string CustomerName { get; set; }

        [Display(Name = "Country")]
        public string CountryName { get; set; }

        [Display(Name = "State / Province / Region")]
        public string RegionName { get; set; }
    }
}
~~~

Note that we're using data annotations in the view model to provide the field labels to display on the view.

When the repository method populates this model it combines data from the **Customers** table with **CountryNameEnglish** from the **Country** table and **RegionName** from the **Region** table. In this way the view model can present information that is more helpful to the user than the index values for country and region from the **Customers** table.

### Order/Index view

The simple list of orders for a customer shows the customer information and the order number and date as read-only fields, and the purchase order / description as an editable field. By changing values in the editable field and saving we can see how model binding works when doing HttpPost actions.

![BlipBinding Order/Index view](https://raw.githubusercontent.com/pluralsight/guides/master/images/b6ffc2ac-a05e-4083-8a37-087a170ac5e7.png)

*BlipBinding solution, Blip.Web project, Order/Index view*

In this view we're presenting information in a hierarchical structure. At the top level is the customer information. Underneath that is the list of orders for the customer.

Let's see how the data is presented in code.

#### Blip.Web\Views\Order\Index.cshtml

For the top-tier data, pertaining to the customer, the fields are composed in a very standard way:

~~~csharp
        <div class="form-group">
            @Html.LabelFor(model => model.CustomerName, new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.CustomerName, new { htmlAttributes = new { @class = "form-control", @readonly = "readonly" } })
            </div>
        </div>
~~~

Note that we're using the `EditorFor` HtmlHelper to let the Razor engine determine the correct type of HTML element for the data type. We're also applying the `form-control` CSS class to be sure the control picks up the appropriate styling. The field is changed from an editable textbox to a display-only field with the application of the `@readonly` HTML attribute.

For the second tier data, the list of orders, we're looping through the records in the view model. But in this case we're **not** using a `foreach` loop and we're **not** using the `EditorFor` HtmlHelper. We'll look at the reasons for theses choices in more detail in the section on saving data.

~~~csharp
@if (Model.Orders != null)
{
    for (var i = 0; i < Model.Orders.Count(); i++)
    {
        <tr>
            @Html.HiddenFor(x => Model.Orders[i].CustomerID)
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].OrderID, new { @class = "form-control", @readonly = "readonly" })
            </td>
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].OrderDate, new { @class = "form-control", @readonly = "readonly" })
            </td>
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].Description, new { @class = "form-control" })
            </td>
        </tr>
    }
}
~~~
You'll also note that we're using a `for` loop with a counting variable rather than a `foreach` loop. **This is crucial to getting binding to work for the 'HttpPost' action, as we'll see soon.**

#### Blip.Entities\Orders.ViewModels\CustomerOrdersDisplayViewModel.cs

The view model for customer orders reflects the hierarchical structure of the view shown above. It assembles the display information about the customer from the **Customers**, **Countries**, and **Regions** tables and includes a property that is a collection of `OrderDisplayViewModel` entities.

~~~csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace Blip.Entities.Orders.ViewModels
{
    public class CustomerOrdersListViewModel
    {
        [Display(Name = "Customer Number")]
        public Guid CustomerID { get; set; }

        [Display(Name = "Customer Name")]
        public string CustomerName { get; set; }

        [Display(Name = "Country")]
        public string CountryNameEnglish { get; set; }

        [Display(Name = "Region")]
        public string RegionNameEnglish { get; set; }

        public List<OrderDisplayViewModel> Orders { get; set; }
    }
}
~~~

Let's take a look at the class that composes the `Orders` collection.

#### Blip.Entities\Orders.ViewModels\OrderDisplayViewModel.cs

Note that each entity in `OrderDisplayViewModel` is linked to the associated customer in `CustomerOrdersListViewModel`. When we transpose the entities into the view model structure we need to preserve the relationship between the entities (and the tables in the database).

~~~csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace Blip.Entities.Orders.ViewModels
{
    public class OrderDisplayViewModel
    {
        public Guid CustomerID { get; set; }

        [Display(Name = "Order Number")]
        public Guid OrderID { get; set; }

        [Display(Name = "Order Date")]
        public DateTime OrderDate { get; set; }

        [Display(Name = "PO / Description")]
        public string Description { get; set; }
    }
}
~~~

Note also that there are no `virtual` properties in either of these classes to provide navigation between entities. The view models serve the functional purpose of the view and are uncoupled from the entity relationships of the classes and the database tables. Accordingly, when two view models are used together they reflect the relationship(s) between the view models, rather than the entities from which their data is drawn.

The repository methods take care of transposing the data from the structure of the entities to the structure of the view models and back again.

### OrdersController action for Index HttpGet

By using MVVM and the repository design pattern, we can make our controller actions succinct and provide separation of concerns between the presentation layer, business logic, and data. We can see that in action in the controller action that populates the **Order/Index** view.

#### Blip.Web\Controllers\OrderController.cs

~~~csharp
using System;
using System.Net;
using System.Web.Mvc;
using Blip.Data.Orders;
using Blip.Entities.Orders.ViewModels;

namespace BlipProjects.Controllers
{
    public class OrderController : Controller
    {
        // GET: Order
        public ActionResult Index(string customerid)
        {
            if(!String.IsNullOrWhiteSpace(customerid))
            {
                if (Guid.TryParse(customerid, out Guid customerId))
                {
                    var repo = new OrdersRepository();
                    var model = repo.GetCustomerOrdersDisplay(customerId);
                    return View(model);
                }
            }
            return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
        }
...
~~~

All that this controller action needs to do when passed a `CustomerID` from the **Customer/Index** view is pass that value to the appropriate repository method and take the resultant data model, an instance of the `CustomerOrdersListViewModel` class, and pass it to the view.

## Saving data

Saving data using default binding can be a tricky process -- the correct approach is not well-documented. This is also a situation where the code fails silently. Developers will see their web pages being populated with data correctly, but the values won't show up in the model when it arrives at the controller action for HttpPost.

To better understand this, we're first going to take a look at the problem, then show how to code the functionality correctly.

### What **not** to do

In the Razor code for the [list of customers](#Blip.Web\Views\Customer\Index.cshtml) above, we saw that we could populate the list using a `foreach` loop and the `DisplayFor` HtmlHelper method. If we used a `foreach` loop for the list of orders, the `<table>` element would look like this:

~~~html
<table class="table">
    <tr>
        <th>
            @Html.DisplayNameFor(model => model.Orders[0].OrderID)
        </th>
        <th>
            @Html.DisplayNameFor(model => model.Orders[0].OrderDate)
        </th>
        <th>
            @Html.DisplayNameFor(model => model.Orders[0].Description)
        </th>
    </tr>
    @if (Model.Orders != null)
    {
        foreach (var order in Model.Orders)
        {
            <tr>
                @Html.HiddenFor(x => order.CustomerID)
                <td>
                    @Html.DisplayFor(x => order.OrderID)
                </td>
                <td>
                    @Html.DisplayFor(x => order.OrderDate)
                </td>
                <td>
                    @Html.EditorFor(x => order.Description)
                </td>
            </tr>
        }
    }
</table>
~~~

That's nice and concise, and seems to leverage the power of Razor HtmlHelper extension methods to "automagically" generate HTML. The problem is; **_it doesn't work_**.

The HTML produced by the preceding code would look like this:


![Ambiguous element ID's](https://raw.githubusercontent.com/pluralsight/guides/master/images/18f64e46-ca29-42d4-84de-66cb6f392211.png)

*Form elements with ambiguous element names and ID's*

Look at the areas highlighted in yellow. Each row is a separate textbox on the form shown above for the [Order/Index view](#Order/Index-view). Each record has the same value for the `name` and `id` elements: `order.Description`. Without a way to identify the records distinctly, MVC **gives up** and returns `null` for the `Orders` field of the `CustomerOrdersListViewModel` to which the **Order/Index** view is bound.

### Correctly binding collection data

In the Razor code for the [list of orders](#Blip.Web\Views\Order\Index.cshtml) for a specific customer, we used a `for` loop with a local variable index value `i`. The loop looks like this:

~~~csharp
@if (Model.Orders != null)
{
    for (var i = 0; i < Model.Orders.Count(); i++)
    {
        <tr>
            @Html.HiddenFor(x => Model.Orders[i].CustomerID)
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].OrderID, new { @class = "form-control", @readonly = "readonly" })
            </td>
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].OrderDate, new { @class = "form-control", @readonly = "readonly" })
            </td>
            <td>
                @Html.TextBoxFor(x => Model.Orders[i].Description, new { @class = "form-control" })
            </td>
        </tr>
    }
}
~~~

Note the following particulars:

1. The index value `i` appears in the lambda expression for each form element being generated by the loop, for example:

    `(x => Model.Orders[i].OrderID)`

1. The `TextBoxFor` HtmlHelper is used, rather than the more general (and automagic) `DisplayFor` and `EditorFor`.

1. The `OrderID` and `OrderDate` fields are set as `readonly` using the HTML `class` attribute, rather than using `DisplayFor`.

1. Bootstrap textbox styling is applied by adding the `@class = "form-control"` attribute.

The generated HTML looks like this:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/e8436cca-23a3-4616-a6cf-9302d32496e8.png)

*Form elements with distinct name and id attributes*

As the areas highlighted in yellow show, each field has a `name` and `id` attribute that is distinct for each record. The record index gives MVC something to use to bind the form data to the data model.

By setting a breakpoint in the HttpPost controller action for the **Order/Index** view we can see that the new data we entered, "expedite" in the `Description` field of record 0, is being posted back to the server along with the values for the `readonly` fields.


![Visual Studio property inspector](https://raw.githubusercontent.com/pluralsight/guides/master/images/b550aa62-b948-430e-9168-e0e122860ec8.png)

*Visual Studio debugging showing property inspector values for an `OrderDisplayViewModel` entity*

### Posting display data

As we noted above, we're using the `TextBoxFor` HtmlHelper for read-only fields, rather than the `DisplayFor` helper. This is so we can return the data to the controller when the HttpPost event fires (when the **Save** button on the page is pressed).

When MVC generates the HTML for a `DisplayFor` HTML helper, it renders the element as simple text.

For example, a table cell coded like this:

~~~html
<td>
    @Html.DisplayFor(modelItem => item.OrderID)
</td>
~~~

would render like this:

~~~html
<td>
    490dabe1-1570-473a-8331-5f32333b2635
</td>
~~~

That's just straight text, so there's no way for MVC to bind it to the model.

But a table cell for a display-only text field coded like this:

~~~html
<td>
    @Html.TextBoxFor(x => Model.Orders[i].OrderID, new { @class = "form-control", @readonly = "readonly" })
</td>
~~~

would render like this:

~~~html
<td>
    <input name="Orders[0].OrderID" class="form-control" id="Orders_0__OrderID" type="text" readonly="readonly" value="490dabe1-1570-473a-8331-5f32333b2635" data-val-required="The Order Number field is required." data-val="true">
</td>
~~~

As an `<input>` field, this element will be passed back to the controller during the `POST`. Because it has a distinct `id`, the data in this `readonly` field can be bound to the view model just like data from an editable field. The values for `CustomerID` and `OrderID` give us the index values necessary to save the changes to the editable field, `Description`.

Note, also, that while we're displaying `OrderDate` as a `readonly` text field, and thereby returning it during the `POST` event, we don't have to. Only the index values necessary to save the changed data need to be returned in the `POST` event.

In our example, the `OrderID` field is displayed, but the `CustomerID` field is included in the form using the `HiddenFor` HtmlHelper. As coded, it looks like this:

~~~html
@Html.HiddenFor(x => Model.Orders[i].CustomerID)
~~~

And the HTML rendered by it looks like this:

~~~html
<input name="Orders[0].CustomerID" id="Orders_0__CustomerID" type="hidden" value="f8214550-69f6-4089-b58a-2de2d4ab01c8" data-val-required="The CustomerID field is required." data-val="true">
~~~

Aside from being hidden on the HTML served to the client, this is a full-featured data element that is bound to the view model received by the `HttpPost` controller action for the **Index** view. Using `HiddenFor` is a convenient way of keeping form layout simple while including all the data necessary for identifying the records to be updated during a `POST`.

[*The astute reader may have realized that in our case study the `CustomerID` field isn't necessary to save changes to an `Order` object. Because `OrderID` is a `GUID` it inherently provides a unique identifier for an individual order.*]

### HttpPost controller actions

By using the repository design pattern we can keep our controller actions simple. In the case of the `HttpPost` action for the **Order/Index** view, all we need to do is validate `CustomerOrdersListViewModel` and pass it to the appropriate repository method.

#### Blip.Web\Controllers\OrdersController.cs

~~~csharp
...
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Index(CustomerOrdersListViewModel model)
{
    if (ModelState.IsValid)
    {
        if (model.Orders != null)
        {
            var repo = new OrdersRepository();
            repo.SaveOrders(model.Orders);
        }
        return View(model);
    }
    return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
}
...
~~~

### Save repository method

The repository method associated with our order list view can also be simple. All we need to do is find the appropriate records in the **Orders** table and update them.

#### Blip.Data\Orders\OrdersRepository.cs

~~~csharp
...
public void SaveOrders(List<OrderDisplayViewModel> orders)
{
    if (orders != null)
    {
        using (var context = new ApplicationDbContext())
        {
            foreach (var order in orders)
            {
                var record = context.Orders.Find(order.OrderID);
                if (record != null)
                {
                    record.Description = order.Description;
                }
            }
            context.SaveChanges();
        }
    }
}
...
~~~

## More information

If you want to dive deeper into the topics discussed in this guide, the following is a curated list of resources.

### Related PluralSight training classes

[PluralSight](https://www.pluralsight.com) offers a number of courses on the topics mentioned in this guide. The following are some suggestions organized by technology:

| **Technology** | **Course(s)** |
| :--- | :--- |
| ASP.NET, EF, View Models | [Best Practices in ASP.NET: Entities, Validation, and View Models](https://app.pluralsight.com/library/courses/aspdotnet-bestpractices-models/table-of-contents)
| ASP.NET | [Building Applications with ASP.NET MVC 4](https://app.pluralsight.com/library/courses/mvc4-building/table-of-contents)
| C# | [C# Fundamentals with Visual Studio 2015](https://app.pluralsight.com/library/courses/c-sharp-fundamentals-with-visual-studio-2015/table-of-contents)
| Entity Framework | [Getting Started with Entity Framework 6](https://app.pluralsight.com/library/courses/entity-framework-6-getting-started/table-of-contents)]
| MVC | [ASP.NET MVC 5 Fundamentals](https://app.pluralsight.com/library/courses/aspdotnet-mvc5-fundamentals/table-of-contents)

### Case study code on GitHub

The complete Visual Studio solutions described in this guide are available on GitHub:

| **Project** | **Repo** |
| :--- | :--- |
| BlipBinding| [https://github.com/ajsaulsberry/BlipBinding](https://github.com/ajsaulsberry/BlipBinding) |

You can fork the projects, run the code, and experiment on your own.

Note that the sample project is not intended to be a real life case study or production code; it exists to illustrate the topics covered in this guide.

### Other resources

> Disclaimer: HackHands and the author of this Guide are not responsible for the content, accuracy, or availability of 3rd party resources.

[Microsoft Entity Framework Documentation](https://msdn.microsoft.com/en-us/library/ee712907(v=vs.113).aspx)  
This page is the jumping off point for a variety of resources on EF.

[Microsoft: The Repository Pattern](https://msdn.microsoft.com/en-us/library/ff649690.aspx)  
This page provides a summary of the benefits of the repository pattern.

---
If you found this guide informative or interesting, please hit the favorites button at the top of the page! Feel free to drop comments and feedback in the discussion section below. Thanks for reading!