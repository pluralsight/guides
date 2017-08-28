# Introduction

### Skill levels

| **Technology** | **Skill Level** |
| :--- | :--- |
| ASP.NET | Beginner
| C# | Beginner
| MVC | Beginner

### Purpose

This guide will demonstrate and explain how you can use ASP.NET MVC to easily create HTML pages with multiple submit buttons, using as little code as possible, and by leveraging the MVC default model binding and controller actions. No JavaScript is required for these techniques.

### Structure

1. For developers looking for a quick answer, we will begin with examples of the code required in the Razor view and Controller for two multibutton techniques.

2. Next we'll take a closer implementation details, security considerations, and ways to pick the best technique.

3. We'll conclude with some suggested resources for learning more about the topics discussed.

### Scope

This guide will focus on the ASP.NET MVC default model binding and controller actions. JavaScript techniques will be covered in a separate guide.

### Prerequisites

You should have:

* a basic understanding of ASP.NET MVC, especially [model binding](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding)
* access to the development tools required to implement ASP.NET MVC solutions.

If you want to try out the examples or see a complete working demonstration of both techniques, you can fork or download the accompanying sample project from [GitHub](https://github.com/ajsaulsberry/BlipCo).

---

# Implementation

*Here are two good ways to implement multiple submit buttons, focusing on just the essential code. More detailed information and more elements of the sample code follow.*

## Technique 1: multiple buttons with the same name invoking default controller actions

By setting a couple standard attributes and adding a parameter to the default `HttpPost` controller action (`ActionResult`) it's easy to get an HTML form to respond differently to different button pushes.

This technique invokes the default ASP.NET MVC controller action for the HttpPost event associated with the HTML page.

### Razor view

The Razor view (.cshtml) contains multiple `<input>` tags of `type="submit"` with the *same* `name` attribute and *different* `value` attributes.

The values (text) of the `value` attributes will be displayed as the button text. *They will also be passed to the controller.*

~~~html
<input type="submit" name="answer" value="Accept" class="btn btn-success" />
<input type="submit" name="answer" value="Decline" class="btn btn-danger" />
<input type="submit" name="answer" value="Defer" class="btn btn-warning" />
~~~

These examples are shown with Bootstrap CSS styling in the `class` attribute to provide appropriate visual guidance to the user. Bootstrap is optional.

### Default controller action

The controller action for the form takes the default binding object as a parameter. In the example below, it's of type `User`.

It also takes a `string` parameter with the same name as the value for the `name` attribute of the `<input>` tags in the associated Razor view ("Index").

The controller action implements a switch statement that tests for the value of the `value` attributes of the `<input>` tags. This is *case-sensitive.*

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Index([Bind(Include = "UserID,Username")] User user, string answer)
{
    if (ModelState.IsValid && !String.IsNullOrWhiteSpace(answer))
    {
        switch (answer)
        {
            case "Accept":
                user.TermsAcceptedOn = DateTime.Now;
                user.TermsStatus = "Accepted";
                break;
            case "Decline":
                user.TermsStatus = "Declined";
                break;
            case "Defer":
                user.TermsStatus = "Deferred";
                break;
            default:
                user.TermsStatus = "Deferred";
                break;
        }

        // Code to save the values for user.Username and user.AcceptedOn to permanent storage could go here.
    }
    return View(user);
}
~~~

## Technique 2: using HTML5 attributes to call different controller methods

Like the first method, this approach still uses multiple `<input>` tags of `type="submit"`, with the same 'name' attribute and different 'value' attributes.

It adds to additional attributes to the `<input>` tag that are implemented in HML5; `formaction` and `formmethod`.

Each button invokes a separate controller action different than the default controller action for the page.

### Razor view (.cshtml)

Like the first technique, this approach uses multiple `<input>` tags with the same `name` attribute values and different values (text) for the `value` attribute. It also adds two attributes which were implemented in HTML5:

`formaction` - specifies what action to take when the button is pressed and

`formmethod` - specifies whether this is an HttpPost or an HttpGet action.

The value of the `formaction` attribute  is the name of the controller action to be called on the button push, either "TermsAccept" or "TermsDecline" in the example.  See the [Form Action](#formaction) section, below, for an explanation of why it's a good idea to use the Razor `@Url.Action()` method to supply the URL.

The value of `formmethod` should be "post" for HttpPost.

The value (text) of the `value` attribute provides the text for the button, as in the first technique.

~~~html
<input type="submit" name="response" value="Accept" formaction=@Url.Action("TermsAccept") formmethod="post" class="btn btn-primary" />
<input type="submit" name="response" value="Decline" formaction=@Url.Action("TermsDecline") formmethod="post" class="btn btn-default" />
~~~

As in the first technique, the Bootstrap styling is optional.

### Controller actions matching `formaction` attribute values

Using the HTML5 `formaction` attribute provides the ability to create separate controller actions for each button. Each of the submit buttons shown above has a controller action that corresponds to the `formaction` value.

A parameter is not needed for the `value` attribute of the buttons created on the page with the `<input>` tags.

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult TermsAccept([Bind(Include = "UserID,Username")] User user)
{
    if (ModelState.IsValid)
    {
        user.TermsAcceptedOn = DateTime.Now;
        user.TermsStatus = "Accepted";
    }
    return View("Index", user);
}

[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult TermsDecline([Bind(Include = "UserID,Username")] User user)
{
    if (ModelState.IsValid)
    {
        user.TermsStatus = "Declined";
    }
    return RedirectToAction("Index", "Home");
}
~~~

This technique provides more control, flexibility, and separation of concerns than using the default controller action for the page does. Some of the capabilities which can be written specifically for the task include:

* Model binding - different fields in the returned data can be bound;
* Error handling - can be written specifically for the action;
* Flow control - each button can have its own flow-of-control, keeping code more comprehensible by limiting the number of return paths from each method.

Note that this technique requires HTML 5 support, so it is not compatible with certain browsers. Any browser that has been updated since 2011 or 2012 supports this functionality. See [caniuse.com](http://caniuse.com/#search=formaction) for browser-specific information.

Also note that, in the example above, the method `TermsDecline` binds the same fields in the `User` object as `TermsAccept`, but it doesn't have to. The second method could also be:

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult TermsDecline()
{
    return RedirectToAction("Index", "Home");
}
~~~

We can see that implementing a controller action for each button is not burdensome as one might expect.

---

# In Depth

Now that you have had the chance to peek under the hood, let's break it down even further.

These are two widely applicable techniques for implementing multiple submit buttons on web pages using the native capabilities of ASP.NET. Minimal code is required and none of it has to be JavaScript.

The following are some developer notes about various aspects of each technique.

### Implementation details

#### [Technique 1: Default controller action](#Technique-1:-multiple-buttons-with-the-same-name-invoking-default-controller-actions)

This approach requires you to keep the `value` attribute of each `<input>` tag in sync with cases of the `switch` statement in your controller. These values are *case-sensitive* when they are strings.

 It also requires you to add a parameter to the controller method that has the same, case-sensitve, name as you give to the group of `<input>` buttons.

 **Numbers are valid parameters:** You can use numeric values for the `value` parameter, in which case you won't be using double-quotes around the value. Your button text will be the number you've assigned to `value` for each button, so this is a use case that's limited to collecting numeric informations through button pushes. (Unless you have a reason for assigning ordinal values to buttons, which isn't very communicative in most situations.) Keep in mind that if you're going to use numbers there are some constraints on the datatypes that are compatible with the [switch statement](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/switch).

#### [Technique 2: Separate controller actions for each button](#Technique-2:-using-HTML5-attributes-to-call-different-controller-methods)

This technique requires you to create separate controller actions for each button. The name of controller action must match the `formaction` attribute for the associated `<input>` tag. The value (text) of the `value` attribute doesn't have to match anything, so it's easier to change the text for the `<input>` button; no corresponding changes have to be made in the controller and the text on the button can be unrelated to the name of the controller action.

It's a good idea to use `@Html.Action(controllerActionName)` to specify the name of the controller action where "controllerActionName" is the actual name of your controller action (like "TermsAccept" in the example above).

~~~html
<input type="submit" name="response" value="Decline" formaction=@Url.Action("TermsAccept") formmethod="post" class="btn btn-default" />
~~~

> Here's why, if you're interested in the details:
>
>When you specify the value for the `formaction` attribute you're providing a URL, which can be a fully qualified path, like `href="http://www.example.com/formresult"` or a relative path like `login/formresult`.
>
>In this technique you're specifying the path to the controller action. In the example above, it would be easy to just plug in the controller action name (`formaction="TermsAccept"`) and most of the time the relative path would resolve correctly.
>
>But when you're using the default page for an MVC route, like "index.cshtml" the relative route alone won't resolve correctly. The Razor HtmlHelper takes care of determining the correct route whether your current path is `/Terms` or `/Terms/Index`.

On the server side, the controller methods that you write for each submit button should be decorated with `[HttpPost]` and `[ValidateAntiForgeryToken]` just like a default POST method would be.

### Security considerations

Both techniques are compatible with the ASP.NET [AntiForgeryToken](https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks), which helps prevent cross-site request forgery attacks. On the client side, be sure to use the `@Html.AntiForgeryToken()` HTML Antiforgery Helper. Decorate the controller action with `[ValidateAntiForgeryToken]` to implement the server side of this safeguard.

In both techniques, binding the controller action only to the elements of the data model you actually need to process helps reduce the surface area for attacks. You often only need to bind to a unique identifyer and the data submitted on the form; data included in the model for to display to the user doesn't need to be bound.

[Technique 1](#Technique-1:-multiple-buttons-with-the-same-name-invoking-default-controller-actions) uses the default HttpPost controller action for the page no matter how many buttons are on the page, so your `switch` statement should always include a `default` case ([see example](#Default-controller-action)). Your other cases will process only the buttons you've identified; a default action can respond to any exegeous inputs that might come your way.

[Technique 2](#Technique-2:-using-HTML5-attributes-to-call-different-controller-methods) is also compatible with [HtmlHelper.Antiforgerytoken](https://msdn.microsoft.com/en-us/library/system.web.mvc.htmlhelper.antiforgerytoken(v=vs.118).aspx). By providing separate controller actions for each button, you can bind data fields more selectively and reduce the attack surface area associated with each action.

With separate methods you can also isolate the code associated with each server response more completely. Error handling and redirects can also be specific to the action.

### Choosing the appropriate technique

The unique features of each technique present relative advantages and disadvantages depending on the specific requirements for the code.

**Technique 1** is likely to be the better choice when:

* the appropriate text for the buttons provides a good value for evaluating which button is pushed (like "Accept" and "Decline")

* the text for the button, and therefore the `value` attribute of the `<input>` tag are unlikely to change

* the `switch` statement, or other logic in the default POST controller can be kept relatively short and simple (since it's a good idea to keep your controller logic short and simple)

* the controller will have one `return` statement for normal execution

* data can be conveyed to the controller in the `value` attribute, form data model, or both

On the other hand, **Technique 2** is the better approach when:

* the text for the buttons is long, contains special characters, is likely to change, or provides a poor set of values to evaluate in the controller;

* the code in the controllers is dissimilar;

* the buttons require different return actions from the controllers;

* data can be conveyed to the controller through the form's data model;

* error handling needs to be different for different buttons.

# Other techniques

#### Multiple buttons with different names

It's possible to create buttons with different names, in which case the HTML in the View looks like this:

~~~html
<input type="submit" name="save" value="Save" />
<input type="submit" name="cancel" value="Cancel" />
~~~

And the default controller looks like this:

~~~csharp
public ActionResult ProcessForm(User user, string save, string cancel)
{
    if(!string.IsNullOrEmpty(save))
        // Do stuff.
    if (!string.IsNullOrEmpty(cancel))
        // Do other stuff.
    return View("SomePage", user);
}
~~~

This is ugly, and it is likely to result in more code. In the end, you probably should avoid this approach. Although your buttons will have different values for the `name` attribute, they will still have different `value` attributes **and** you'll have to create a controller method parameter for each button.

This might make sense if you have two buttons and their names are short, but their text is long, like:

|   **name**   |   **value**   |
| :--- | :--- |
|   save   |   Forward unto the next millenium!   |
|   cancel   |   Cancel   |

But long button text violates most user interface design guidelines, so you probably shouldn't be doing that, either.

Bleh.

#### JavaScript and Ajax

You can implement multiple submit buttons in JavaScript. Pluralsight has excellent guides on learning JavaScript and creating buttons.

You can use Ajax also. However, in that scenario, you will need to implement something to prevent cross-site request forgery (CSRF) attacks.

# More information

Here are some suggested resources if you need to touch up on MVC, dig into the specifics of Razor-enhanced HTML, or learn more about controllers.

> *Disclaimer:* HackHands and the author of this Guide are not responsible for the content, accuracy, or availability of 3rd party resources.

### PluralSight training classes

PluralSight has strong course offerings and complete learning tracks for Microsoft-based technologies, including ASP.NET MVC and C#. Some of the most relevant are:

[Building Applications with ASP.NET MVC 4](https://app.pluralsight.com/library/courses/mvc4-building/table-of-contents) by Scott Allen, updated 8 Nov 2012. Don't let the age of the course dissuade you: the fundamentals have changed little and this is a good launching point if you are new(ish) to MVC.

[ASP.NET MVC 5 Fundamentals](https://app.pluralsight.com/library/courses/aspdotnet-mvc5-fundamentals/table-of-contents) by Scott Allen, updated 5 Nov 2013. See especially Chapter 4, Bootstrap, for info on styling your form and its buttons.

### Extra resources

Microsoft has made a start at a new generation of developer documentation at [docs.microsoft.com](https://docs.microsoft.com) and it includes a quick tutorial:  [Getting Started with ASP.NET MVC 5](https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/introduction/getting-started).

You can get a complete, ASP.NET MVC 5 project that corresponds to the examples in this guide from [GitHub](https://github.com/ajsaulsberry/BlipCo).

______________

I hope you enjoyed this guide. Please leave your comments and feedback in the discussion section below, and don't forget to favorite this guide if you found it informative or interesting!

*Written by A. J. Saulsberry, 14 August 2017.*