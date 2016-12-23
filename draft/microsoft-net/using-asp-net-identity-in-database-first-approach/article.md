## Introduction 
In most of the cases when it comes to Identity configuration in ASP.NET it can be quite confusing. Some developers say that the configuration is easy but other claim that sometimes it can be difficult to set it up, especially if you want to customize some of its properties. When you use code-first approach in Entity Framework, you have full control on your user identity options, but when developers deal with bigger projects, usually they prefer to use table-first approach (first they create the database and then they consume the information in the api and shape it in an appropriate way, so it makes sense when the frontend receives it). 


Imagine that the client of our fictional software company is a huge car manufacturer. They have a lot of shops all around the world, where they sell their products. The first and most important feature of their system should be user-management. It should have different types of users:  Admin, Regional Manager, Shop Manager, Seller and Client. In addition, all of them, except the client, have some secret company code that they can use for access to certain documents related to company’s deals. 

### Setting up the database
After we have defined the main properties a user in our application should have, it is time to start developing the architecture of our system. Since it can scale fast and the database design is important, we will use table-first approach. The first step is to create our database. For this article, I will use SQL Server 2016 in combination with SQL Management Studio. 


![Creating a blank database](https://raw.githubusercontent.com/pluralsight/guides/master/images/b33c22e7-b4bf-4779-a137-a65098a5b2b1.PNG)


### Creating a new project
Now, after we have the blank database, we can set up the asp.net web api project, which is going to consume the information. Execute the following steps to do it:
1.	Create a new project in Visual Studio and choose Blank Solution. Name it CarBusinessSystem.
2.	Add a new web api project to this solution. Name it WebApi. In this way, we can use the template to scaffold our user accounts system. 
After you complete these steps, you should be able to see the following set-up in your Solution Explorer:



![Project foundation](https://raw.githubusercontent.com/pluralsight/guides/master/images/d7da08a8-00dc-465c-a4ba-dac38dbee5ea.PNG)

 
We will have several important files for the purpose of this article.
•	AppStart\IdentityConfig.cs
•	AppStart\Startup.Auth.cs
•	Providers\ApplicationOAuthProvider.cs
•	Models\IdentityModels.cs
•	Controllers\AccountController.cs

#### Linking
Now, after we have our database and our asp project created, we should find a way to link them. In order to achieve this, we will create a DbContext by basing it on a connection string, pointing to our database.  Open the Web.config file and see what happens between the <connectionStrings> for now we have the only the default connection, which points to an instance of LocalDb. We can also notice that the default ApplicationDbContext class in Models\IdentityModels.cs is based on this connection. Our idea here is to create a new context and then base our ```ApplicationUserManager``` on it. 
```
<add name="SystemUsers" connectionString="Data Source=.;Initial Catalog=CarBusinessDb;" providerName="System.Data.SqlClient" />
```
Delete the default connection string and paste this one on its place. 
The next step is to create our own database context, which we can use for storing our users and their properties. In the `Models\IdentityModels.cs` we are going to delete the ApplicationDbContext paste the following code on its place. 
```C#
public class AppUsersDbContext : IdentityDbContext<ApplicationUser>
    {
        public AppUsersDbContext()
            : base("SystemUsers", throwIfV1Schema: false)
        {
        }

        public static AppUsersDbContext Create()
        {
            return new AppUsersDbContext();
        }
    }
```
As you can see, we use the new connection we have created in the Web.config file for the new context passing its name as a string. 
Once we have the connection between the database and the asp project, we should configure the built in ApplicationUserManager, so it is going to use this context instead of the default one, which we have already deleted. A quick look on both ```UserStore``` and ```ApplicationUserManager``` classes: 
 ```csharp
 namespace Microsoft.AspNet.Identity.EntityFramework
{
    //
    // Summary:
    //     EntityFramework based user store implementation that supports IUserStore, IUserLoginStore,
    //     IUserClaimStore and IUserRoleStore
    //
    // Type parameters:
    //   TUser:
    public class UserStore<TUser> : UserStore<TUser, IdentityRole, string, IdentityUserLogin, IdentityUserRole, IdentityUserClaim>, IUserStore<TUser>, IUserStore<TUser, string>, IDisposable where TUser : IdentityUser
    {
        //
        // Summary:
        //     Default constuctor which uses a new instance of a default EntityyDbContext
        public UserStore();
        //
        // Summary:
        //     Constructor
        //
        // Parameters:
        //   context:
        public UserStore(DbContext context);
    }
}
 ```
 ```csharp
 namespace WebApi
{
    // Configure the application user manager used in this application. UserManager is defined in ASP.NET Identity and is used by the application.

    public class ApplicationUserManager : UserManager<ApplicationUser>
    {
        public ApplicationUserManager(IUserStore<ApplicationUser> store)
            : base(store)
        {
        }

        public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context)
        {
            ///Calling the non-default constructor of the UserStore class
            var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
            
            /// Rest of the class ...
        }
    }
}
```
Shows us that the ApplicationUserManager calls the constructor of the UserStore, which accepts a DbContext and then it uses exactly this connection to store the users data. So, here it is enough just to pass our custom context as a parameter, when the ApplicationUserManager calls the  UserStore constructor.  Substitute the manager variable with the following code:
```csharp
var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<AppUsersDbContext>()));
```
The next step is to initialize our context each time our application starts, in this way, we will be sure that the user manager and the context use the same instance. Go to the ```AppStart\Startup.Auth.cs``` file, where we can see that currently, only the deleted ```ApplicationDbContext``` is initialized. 

```
public void ConfigureAuth(IAppBuilder app)
        {
            // Configure the db context and user manager to use a single instance per request
            app.CreatePerOwinContext(ApplicationDbContext.Create);
            app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);

            // Enable the application to use a cookie to store information for the signed in user
            // and to use a cookie to temporarily store information about a user logging in with a third party login provider
            app.UseCookieAuthentication(new CookieAuthenticationOptions());
            app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);

            /// Rest of the class ...
        }
```
 
Change the ```app.CreatePerOwinContext(ApplicationDbContext.Create);``` line with the following code:
```
app.CreatePerOwinContext(AppUsersDbContext.Create);
```
Now, we have set-up our API, in a way to use the newly created database, when it comes to storing users. Next thing we should do is to create the actual tables, where this data will be stored, so close Visual Studio for a while and open the database in your MSMS. 


### Creating Tables for Storing User Data
To know what kind of columns, we should create in our table, we should read the default IdentityUser class, because as we can see the ApplicationUser inherits from it, without adding any additional properties. A quick look at the [documentation](https://aspnetidentity.codeplex.com/SourceControl/latest#src/Microsoft.AspNet.Identity.EntityFramework/IdentityUser.cs) of this class at codeplex can show us that the default user a lot of properties, which I am not going to list here, but you can read them together with a short description explaining why each of them is there by following the provided link. For the purpose of this tutorial I will post a TSQL script and then explain to you how to execute it, so it will create the needed tables. But first let us define how our ApplicationUser  class will look and what extension properties we want to add on the top of the ApplicationUser  class. I will use the following properties on the top of the inherited entity:
•	CompanySecretCode (the code mentioned in the beginning of the article)
•	RegionName (the name of the region, where this user works)

The company secret code will be a sequence of six digits and the name of the region will be string. 



