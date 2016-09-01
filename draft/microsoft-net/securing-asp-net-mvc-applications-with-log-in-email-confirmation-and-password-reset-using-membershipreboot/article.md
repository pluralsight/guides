Security is often a discussed topic and having an easy way to manage users accounts and authentication is something we've always wanted. Thanks to the asp.net team, ways of doing this keep getting better everyday, and we can see, over the years, the libraries been released for this purpose. We have moved from ASP.NET Membership, to SimpleMembership, and now ASP.NET Identity. I can look back early on in my career and see myself struggling trying to extend or add extra configuration to the then ASP.NET Membership. With this evolution, we have also moved from Role based authorization to Claims based authorization. 

[MembershipReboot (MR)](https://github.com/brockallen/BrockAllen.MembershipReboot) is a Claims-based user account and identity management framework, that makes it easy for you to manage users accounts. It encapsulates important security logic while making it easy for developers to configure and extend. It's features include but not limited to: 

* Extensible templating for email notifications
* Single- or multi-tenant account management
* Account linking with external identity providers (enterprise or social)
* Notification system for account activity and updates
* Claims-aware user identities
* Flexible account storage design (you can choose to store your data in a relational or non-relational data store)

In this tutorial, I am going to show you how to secure your ASP.NET MVC apps using this library. I will show how to set it up, create a user account, enable account/email verification, and allow password reset. I will be doing this with a new project, but for existing apps, you can start from the section that install the needed nuget packages.

# Let's Begin

## Create a new MVC Web Application

1. Select File > New Project > ASP.NET Web Application. Enter a name and click `OK`. 
2. Select the MVC template.
3. Click on the `Change Authentication`button and choose **No Authentication** and then click __OK__
4. Now we have the project set-up and we're ready to roll :+1 

## Install the nuget packages
Next on is to install the following nuget packages:
* > __Install-Package BrockAllen.MembershipReboot.WebHost__

This is the Web Host package that performs the job of issuing cookies to track the logged in user, performing two-factor authentication, and the ApplicationInformation details. It has dependencies on `BrockAllen.MembershipReboot`(which is the core of the framework), therefore, these packages where installed when we ran the above command.

 * > __Install-Package BrockAllen.MembershipReboot.Ef __

This is the Entity Framework persistence implementation, responsible for persisting data to a SQL datastore. This is because it has a flexible storage design, where you can choose to store your data in either SQL or NoSQL database, and for this tutorial, I've chosen to work with a SQL datastore using this package.

* > **Install-Package SimpleInjector.Integration.Web.Mvc**
* > **Install-Package SimpleInjector.Extensions.ExecutionContextScoping**


Simple Injector ASP.NET MVC Integration. We will be using SimpleInjector as our Dependency Injector (DI) container. 

## Replace Forms Authentication with Session Authentication Module (SAM)
Because MR is claims-based, we need to add settings for claims-based identity and Session Authentication Module (SAM) to the web.config. To do this, we need to:
* First add references to __System.IdentityModel__ and __System.IdentityModel.Services__.
* Add some `<configSections>` elements to the web.config 

```xml
<section name="system.identityModel" type="System.IdentityModel.Configuration.SystemIdentityModelSection, System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />
<section name="system.identityModel.services" type="System.IdentityModel.Services.Configuration.SystemIdentityModelServicesSection, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=B77A5C561934E089" />

```
* Add configuration section for federation configuration

```xml
<system.identityModel.services>
<federationConfiguration>
<cookieHandler requireSsl="false" /> <!-- // set to true before deploying to production server // -->
</federationConfiguration>
</system.identityModel.services>

```

* Under `<system.webServer>.<modules>` add the SAM (session authentication module) to the http modules list:

```xml
<add name="SessionAuthenticationModule" type="System.IdentityModel.Services.SessionAuthenticationModule, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" preCondition="managedHandler" />
```

## Add settings for connecting to the database
MembershipReboot allows for flexible storage of the user account data, like I mentioned earlier. There is an EntityFramework package that implements the appropriate persistence interface, which we have installed already. For this, we need to set the connection string value, which by default, it looks for a connectionString named `MembershipReboot`. I'm going to use this default for this tutorial. Adding this in your web.config would look similar to:
```xml
<add name="MembershipReboot" connectionString="Data Source=(LocalDb)\v11.0;Initial Catalog=MembershipReboot;Integrated Security=True" providerName="System.Data.SqlClient" />
```

## Configure email and security settings for MR
There are three important classes used for configuring MembershipReboot. They are:

* SecuritySettings
> The SecuritySettings class contains settings to configure MembershipReboot for certain security settings (such as account lockout duration and if email verification is required).

* ApplicationInformation
> ApplicationInformation is a class that lets MembershipReboot know the name of the application and what URLs the application provides for various account related functions. This information is mainly used when sending emails or SMSs to the user.

* MembershipRebootConfiguration
> MembershipRebootConfiguration is the main configuration class used by the UserAccountService. It contains the SecuritySettings and provides extensibility points for adding any custom logic related to validation or event notifications when account related data changes.

#### Configure the security settings for the application.
These values can be set via a custom configuration element in the configuration file, using the `SecuritySettings` class which then will passed to the MembershipRebootConfiguration class or alternatively, when creating the `MembershipRebootConfiguration` class. I'm going to use the last option here. To do that, we  add a class that returns an instance of `MembershipRebootConfiguration` class, with the desired settings. 

```csharp
    public class MembershipRebootConfig
    {
        public static MembershipRebootConfiguration Create()
        {
            var config = new MembershipRebootConfiguration();
            config.RequireAccountVerification = true;
            config.EmailIsUsername = false;

            return config;
        }
    }
```

Above, I set the `RequireAccountVerification` property to true, because I want accounts to be verified. I left the other settings to their default. [This page](https://github.com/brockallen/BrockAllen.MembershipReboot/wiki/Security-Settings-Configuration) shows other setting options you can set/configure and also their default value.

#### Add settings for sending emails
Email notifications are handled as part of MR's eventing system. As operations occur on the `UserAccount` class, an event is raised to indicate the operation. There is an event handler class for handling email events from the package we installed. It is called `EmailAccountEventsHandler`. The registration of this class is performed on the MembershipRebootConfiguration class via the `AddEventHandler` API. To instantiate the `EmailAccountEventsHandler` class, an instace of `EmailMessageFormatter` is needed. `EmailMessageFormatter` class reads templated text files that are embedded within the MembershipReboot assembly itself, and the `EmailAccountEventsHandler` class reads text  that will be contained in the email from this class. This text is customizable by either deriving from EmailMessageFormatter or defining a new class that implements IMessageFormatter. 

When sending emails, the `EmailMessageFormatter` needs to embed URLs back into the application. These endpoints are expected to be implemented by the application itself. To inform the `EmailMessageFormatter` of the URLs, the `AspNetApplicationInformation` class or  the `OwinApplicationInformation` (used for owin based applications) can be used. It allows indicating relative paths for the various URLs. If more customization is required, the base `ApplicationInformation` class can be used instead.

For our solution, we're going to use the `AspNetApplicationInformation` class, so we need to update the code that creates a new instance of the `MembershipRebootConfiguration` class. Below is the update made to this class:

```csharp
    public class MembershipRebootConfig
    {
        public static MembershipRebootConfiguration Create()
        {
            var config = new MembershipRebootConfiguration();
            config.RequireAccountVerification = true;
            config.EmailIsUsername = false;

            var appInfo = new AspNetApplicationInformation(
                "Test",
                "Test Hack.guide tutorials",
                "Account/Login/",
                "Account/ConfirmEmail/",
                "UserAccount/CancelRegistration/",
                "Account/ConfirmPasswordReset/");

            var emailFormatter = new EmailMessageFormatter(appInfo);
            config.AddEventHandler(new EmailAccountEventsHandler(emailFormatter));

            return config;
        }
    }
```

On instantiating the `AspNetApplicationInformation` class, we added urls to actions which we will implement shortly. These Urls include links to Login, confirm account, and reset password confirmation. 

For the emails to work, we need to add SMTP settings in the web.config file. Adding this to your web.config will look similar to this:

```xml
<system.net>
    <mailSettings>
      <smtp from="###">
         <network host="###" userName="###" password="#" port="#" />
      </smtp>      
    </mailSettings>
  </system.net>

```

## Configure SimpleInjector
Next up is to configure SimpleInjector to resolve instances of the needed classes of MR in the application, as I'll be using the Dependency Injection (DI) pattern. Any other DI library canbe used but I've chosen to use SimpleInjector for this tutorial. How this services are resolved will  be different in other DI containers based on their own way of configuring services to be injected.

To do this, I will add a class called `SimpleInjectorConfig` in the App_Start folder. I have this class here as I prefer to have my start-up configurations in this file, so it's a matter of preference, and how you do this is up to you. In there I will add a static method with settings for how the needed MR classes will be resolved, and will be called in the Application_Start of the Global.asax.cs. below, is how i've configured this class:

```csharp
    public class SimpleInjectorConfig
    {
        public static void Register()
        {
            // Create the container as usual.
            var container = new Container();
            container.Options.DefaultScopedLifestyle = new WebRequestLifestyle();

            // Register types:
            container.RegisterSingleton<MembershipRebootConfiguration>(MembershipRebootConfig.Create);
            container.RegisterPerWebRequest<DefaultMembershipRebootDatabase>(() => new DefaultMembershipRebootDatabase());
            container.Register<UserAccountService>(() => new UserAccountService(container.GetInstance<MembershipRebootConfiguration>(), container.GetInstance<IUserAccountRepository>()));//Or make it InstancePerHttpRequest
            container.Register<AuthenticationService, SamAuthenticationService>();

            var defaultAccountRepositoryRegistration =
                Lifestyle.Scoped.CreateRegistration<DefaultUserAccountRepository>(container);

            container.AddRegistration(typeof(IUserAccountQuery), defaultAccountRepositoryRegistration);
            container.AddRegistration(typeof(IUserAccountRepository), defaultAccountRepositoryRegistration);


            // This is an extension method from the integration package.
            container.RegisterMvcControllers(Assembly.GetExecutingAssembly());
            // This is an extension method from the integration package as well.
            container.RegisterMvcIntegratedFilterProvider();

            container.Verify();
            //Set dependency resolver for MVC
            DependencyResolver.SetResolver(new SimpleInjectorDependencyResolver(container));
        }
    }

```

Now update your Global.asax.cs class to call the `SimpleInjectorConfig.Register()` method and also add code to tell EntityFramework to add/create the needed tables for MR when the application starts. 

```csharp
protected void Application_Start()
{
    Database.SetInitializer(new MigrateDatabaseToLatestVersion<DefaultMembershipRebootDatabase, BrockAllen.MembershipReboot.Ef.Migrations.Configuration>());

    SimpleInjectorConfig.Register();

    AreaRegistration.RegisterAllAreas();
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    RouteConfig.RegisterRoutes(RouteTable.Routes);
    BundleConfig.RegisterBundles(BundleTable.Bundles);
}
```

Now we have all this set-up. It's time to move on to the real deal.

## Allow users create an account
We need to provide a means for the users to register/create an account. To do this:
* We need add a class that to accept user input. So let's create a class called `RegisterInputModel`

```csharp
public class RegisterInputModel
    {
        [Required]
        public string Username { get; set; }

        [Required]
        [EmailAddress]
        public string Email { get; set; }

        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        [Required]
        [System.ComponentModel.DataAnnotations.Compare("Password", ErrorMessage = "Password confirmation must match password.")]
        [DataType(DataType.Password)]
        public string ConfirmPassword { get; set; }
    }
    
```

* Then Add a conroller and add actions to handle the request for creating an account, confirming their email, and cancel the account creation request. Create a controller called `Account` and add the following code:

```csharp
public class AccountController : Controller
    {
        private readonly AuthenticationService _authenticationService;
        private readonly UserAccountService _userAccountService;

        public AccountController(AuthenticationService authenticationService)
        {
            _authenticationService = authenticationService;
            _userAccountService = authenticationService.UserAccountService;
        }

        public ActionResult Register()
        {
            return View(new RegisterInputModel());
        }

        [ValidateAntiForgeryToken]
        [HttpPost]
        public ActionResult Register(RegisterInputModel model)
        {
            if (ModelState.IsValid)
            {
                try
                {
                    _userAccountService.CreateAccount(model.Username, model.Password, model.Email);

                    return View("Success", model);
                }
                catch (ValidationException ex)
                {
                    ModelState.AddModelError("", ex.Message);
                }
            }
            return View(model);
        }

        public ActionResult Cancel(string id)
        {
            try
            {
                _userAccountService.CancelVerification(id);
                return View("Cancel");
            }
            catch (ValidationException ex)
            {
                ModelState.AddModelError("", ex.Message);
            }
            return View("Error");
        }
    }
    
```

In this class, I have method that handle creating and account and also an action method to cancel/delete an account when the user who get's the email feels they didn't create an account on the application. So this delete's the account from the database. Also, you noticed that we used the `UserAccountService` class. This class has methods to create an account, delete an account, and other extra methods which will will see in this tutorial.

I have omitted show the code I used for the views. You should add this to your code and design the UI how you want it.

Now when the user registers, an email is sent to them. Now, we need to add logic to verify their account. 

### Verify Account
Now we need to add logic for account verification.

* Create a class called `ChangeEmailFromKeyInputModel` 

```csharp
public class ChangeEmailFromKeyInputModel
    {
        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        [HiddenInput]
        public string Key { get; set; }
    }
    
```

* Add the following code to the `Account` controller

```csharp
[AllowAnonymous]
        public ActionResult ConfirmEmail(string id)
        {
                var vm = new ChangeEmailFromKeyInputModel();
                vm.Key = id;
                return View("ConfirmEmail", vm);
        }

        [AllowAnonymous]
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult ConfirmEmail(ChangeEmailFromKeyInputModel model)
        {
            if (ModelState.IsValid)
            {
                try
                {
                    UserAccount account;
                    _userAccountService.VerifyEmailFromKey(model.Key, model.Password, out account);

                    _authenticationService.SignIn(account);
                    return RedirectToAction("ConfirmSuccess");
                }
                catch (ValidationException ex)
                {
                    ModelState.AddModelError("", ex.Message);
                }
            }

            return View("ConfirmEmail", model);
        }
        
```

Also, I would expect you create the views for this classes on your own, as they are simply input forms and success pages.

## Reset Password
Next up, a common feature required in all apps, is the reset password feature, for resetting password to an account, just in case it's forgotten. For this:

* Add a class for accepting the email of the account to reset

```
public class PasswordResetInputModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; }
    }
    
```

* And another class for accepting the new password to update the account

```
public class ChangePasswordFromResetKeyInputModel
    {
        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        [Required]
        [System.ComponentModel.DataAnnotations.Compare("Password", ErrorMessage = "Password confirmation must match password.")]
        [DataType(DataType.Password)]
        public string ConfirmPassword { get; set; }

        [HiddenInput]
        public string Key { get; set; }
    }
    
```

* Then, add the following actions to the Account controller

```
public ActionResult RequestPasswordReset()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult RequestPasswordReset(PasswordResetInputModel model)
        {
            if (ModelState.IsValid)
            {
                try
                {
                    var account = _userAccountService.GetByEmail(model.Email);
                    if (account != null)
                    {
                        _userAccountService.ResetPassword(model.Email);
                        return View("ResetSuccess");
                    }
                    else
                    {
                        ModelState.AddModelError("", "Invalid email");
                    }
                }
                catch (ValidationException ex)
                {
                    ModelState.AddModelError("", ex.Message);
                }
            }
            return View(model);
        }

        public ActionResult ConfirmPasswordReset(string id)
        {
            var vm = new ChangePasswordFromResetKeyInputModel()
            {
                Key = id
            };
            return View(vm);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult ConfirmPasswordReset(ChangePasswordFromResetKeyInputModel model)
        {
            if (ModelState.IsValid)
            {
                try
                {
                    UserAccount account;
                    if (_userAccountService.ChangePasswordFromResetKey(model.Key, model.Password, out account))
                    {
                        if (account.IsLoginAllowed && !account.IsAccountClosed)
                        {
                            _authenticationService.SignIn(account);
                            return RedirectToAction("Index", "Home");
                        }

                        return View("PasswordUpdated");
                    }
                    else
                    {
                        ModelState.AddModelError("", "Error changing password. The key might be invalid.");
                    }
                }
                catch (ValidationException ex)
                {
                    ModelState.AddModelError("", ex.Message);
                }
            }
            return View();
        }
        
```

With this, we have the reset password feature rolled out, and now we want to allow the users to Login to the application.

## Allow users Login and Logout
And for the last part we add logic to allow users log in and out of the system. To start with, add a class for accepting users name and password

```
public class LoginInputModel
    {
        [Required]
        public string Username { get; set; }
        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        public bool RememberMe { get; set; }

        [ScaffoldColumn(false)]
        public string ReturnUrl { get; set; }
    }
    
```

Then add the follwoing action methods:

```
[AllowAnonymous]
        public ActionResult Login()
        {
            return View(new LoginInputModel());
        }
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Login(LoginInputModel model)
        {
            if (ModelState.IsValid)
            {
                UserAccount account;
                if (_userAccountService.Authenticate(model.Username, model.Password, out account))
                {
                    _authenticationService.SignIn(account, model.RememberMe);

                    if (Url.IsLocalUrl(model.ReturnUrl))
                    {
                        return Redirect(model.ReturnUrl);
                    }

                    return RedirectToAction("Index", "Home");
                }

                ModelState.AddModelError("", "Invalid Email or Password");
            }

            return View(model);
        }

        public ActionResult Logout()
        {
            if (User.Identity.IsAuthenticated)
            {
                _authenticationService.SignOut();
                return RedirectToAction("Login");
            }

            return RedirectToAction("Index", "Home");
        }
        
```

For the Login, we have to validate the username and password and we do this with the `UserAccountService.Authenticate()` method. This method has various overloads but the one I've chosen checks against the username and passowrd. If this check is valid, we log the user in by calling `AuthenticationService.SignIn()` method, and to log out, we call the `SignOut()` method on that same class. The AuthenticationService class is a helper that bridges the gap between MembershipReboot and the running web application. It will perform the work of issuing cookies to track the logged in user. It also provides the assistance for performing two-factor authentication and supporting external login providers.

With all these, I hope to have covered the basic needs of securing web application with MembershipReboot and have explained a little about the framework. In a future post, I will show more use cases like two factor SMS authentication. If you have any questions or problems running this, leave a comment, or open an issue on the `GitHub` Repository.