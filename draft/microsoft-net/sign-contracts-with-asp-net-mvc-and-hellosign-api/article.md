
This guide is designed to show you how to work with HelloSign API to send and sign documents via ASP.NET MVC.

HelloSign is a service that allows you to sign documents, and can be used to legalize agreements, preparing contracts, among others.

For this tutorial, we will use:
- ASP.NET MVC 5
- HelloSign SDK (C#)
 
## 1. Configure tools

First we need to create a project with Visual Studio (ideally version 2015). We selected a ASP.NET project (in this case we do not need larger configurations).

Imagen

Then we install the HelloSign SDK through NuGet:

```c#
Install-Package HelloSign
```

We create a controller called BaseController, which we used to store the API Key and Client ID

```c#
public class BaseController : Controller
{
    protected string HelloSignAPIKey = ConfigurationManager.AppSettings["HelloSignAPIKey"];
    protected string HelloSignClientID = ConfigurationManager.AppSettings["HelloSignClientID"];
}
```

Then we create a controller called SignController, that inherits from our base controller. In our controller we will create a GET action called SendDocument:

```c#
public ActionResult SendDocument()
{
    SendDocumentViewModel model = new SendDocumentViewModel();
    return View(model);
}
```



