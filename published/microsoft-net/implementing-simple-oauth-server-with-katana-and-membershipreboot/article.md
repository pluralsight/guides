In my [previous post about .NET](http://tutorials.pluralsight.com/microsoft-net/securing-asp-net-mvc-applications-with-log-in-email-confirmation-and-password-reset-using-membershipreboot), I covered securing and managing user accounts with MembershipReboot in ASP.NET MVC. In this post, I'll walk you through implementing the [Resource Owner Password Credential Grant type](https://tools.ietf.org/html/rfc6749#section-4.3) in the ASP.NET web API. 

If you're reading this, I'll assume that you have a fair amount of knowledge about the Owin (Open Web Interface) middleware in ASP.NET and that you understand OAuth2 and Katana well. Take a look [here](http://www.codeproject.com/Articles/864725/ASP-NET-Understanding-OWIN-Katana-and-the-Middlewa) if you need a refresher on any of the above.

I won't cover these topics, but I'll provide code which you can add to your ASP.NET project. That way, you will be able to use this grant type for securing access to your service specifically.

The Resource Owner Password Credential Grant type is suitable in cases where the resource owner (user's of the application who own a specific resource) has a trust relationship with the client of the web service we want to protect (the application that requests access to a resource on behalf of the resource owner). For more on the importance of grant types, check out [the OAuth2 page](http://bshaffer.github.io/oauth2-server-php-docs/overview/grant-types/). 

To get started, let's install the following nuget packages. 

## Install Nuget packages

* First install SimpleInjector, a dependency injection container

`Install-Package SimpleInjector.Integration.WebApi`

* Then install the packages for MembershipReboot. 

`Install-Package BrockAllen.MembershipReboot.Owin`

`Install-Package BrockAllen.MembershipReboot.Ef`

## Configure the MembershipReboot and DI container and register the container to be used to resolve dependencies at runtime

Next up is to configure the frameworks we added previously. I'll start by adding a connection string to the database that stores the user credentials in the `web.config`file.

```
<add name="MembershipReboot" connectionString="Data Source=(LocalDb)\v11.0;Initial Catalog=MembershipRebootWebAPI;Integrated Security=True" providerName="System.Data.SqlClient" />
```

### Configure MembershipReboot in code
Let's configure MembershipReboot. I usually prefer doing this in code, and having it in a file in the `App_Start` folder. 

Add a file named `MembershipRebootConfig` in the `App_Start` folder, with the following content:

```csharp
using BrockAllen.MembershipReboot;

public class MembershipRebootConfig
{
	public static MembershipRebootConfiguration Create()
	{
		var config = new MembershipRebootConfiguration
		{
			MultiTenant = true,
			RequireAccountVerification = true,
			EmailIsUsername = true,
			AllowLoginAfterAccountCreation = true
		};

		return config;
	}
}

```

Above, I enabled **multiple tenant** because the user accounts will be stored on one tenant, and the API client credentials will be on another. Other settings in the file can be modified as you want. This file added is optional as you can add this same settings in the web.config file.

### Configure SimpleInjector
After this, configure SimpleInjector to resolve MembershipReboot's classes at runtime. Add a new class in the `App_Start`folder called, with the following content in it

``` csharp
public static class SimpleInjectorWebApiInitializer
{
	/// <summary>Initialize the container and register it as Web API Dependency Resolver.</summary>
	public static void Initialize(IAppBuilder app)
	{
		var container = new Container();
		container.Options.DefaultScopedLifestyle = new ExecutionContextScopeLifestyle();

		InitializeContainer(container);

		container.RegisterWebApiControllers(GlobalConfiguration.Configuration);

		container.Verify();

		GlobalConfiguration.Configuration.DependencyResolver =
			new SimpleInjectorWebApiDependencyResolver(container);

        //allow scoped instances to be resolved during an OWIN request
		app.Use(async (context, next) =>
		{
			using (container.BeginExecutionContextScope())
			{
			    context.Environment.SetUserAccountService(() => container.GetInstance<UserAccountService>());
				await next();
			}
		});
	}

	private static void InitializeContainer(Container container)
	{
		System.Data.Entity.Database.SetInitializer(new System.Data.Entity.MigrateDatabaseToLatestVersion<DefaultMembershipRebootDatabase, BrockAllen.MembershipReboot.Ef.Migrations.Configuration>());
		
		container.RegisterSingleton<MembershipRebootConfiguration>(MembershipRebootConfig.Create);
		container.Register<DefaultMembershipRebootDatabase>(() => new DefaultMembershipRebootDatabase(), Lifestyle.Scoped);
		
		var defaultAccountRepositoryRegistration =
			Lifestyle.Scoped.CreateRegistration<DefaultUserAccountRepository>(container);
			
		container.AddRegistration(typeof(IUserAccountQuery), defaultAccountRepositoryRegistration);
		container.AddRegistration(typeof(IUserAccountRepository), defaultAccountRepositoryRegistration);
		container.Register<UserAccountService>(() => new UserAccountService(container.GetInstance<MembershipRebootConfiguration>(), container.GetInstance<IUserAccountRepository>()), Lifestyle.Scoped);
	}
}

```

Above, I've registered MembershipReboot's dependencies, and also added code to make SimpleInjector resolve dependies during an Owin request. 

Open the Owin startup class and call `Initialize`:
```
//Startup.cs
public void Configuration(IAppBuilder app)
{
    SimpleInjectorWebApiInitializer.Initialize(app);
}
```

## OAuth2 token endpoint setup
Next we need to set up the OAuth 2.0 token endpoint to support the Resource Owner Password Credentials Grant type by using the `OAuthAuthorizationServerMiddleware` which comes with the `Microsoft.Owin.Security.OAuth` library. This requires an authorization server which will then be wired up to the Katana pipeline. 

Writing an authorization server using Katana revolves around a class that derives from `OAuthAuthorizationServerProvider`, which contains methods that let us manipulate the OAuth2 protocol. 

To do this, I'll add a class called `MyOAuthAuthorizationServerProvider` which derives from `OAuthAuthorizationServerProvider`and override methods  ValidateClientAuthentication and GrantResourceOwnerCredentials. 

```csharp
public class MyOAuthAuthorizationServerProvider : OAuthAuthorizationServerProvider
{
	public override System.Threading.Tasks.Task ValidateClientAuthentication(OAuthValidateClientAuthenticationContext context)
	{
		string cid, csecret;
		if (context.TryGetBasicCredentials(out cid, out csecret))
		{
			var svc = context.OwinContext.Environment.GetUserAccountService<UserAccount>();
			if (svc.Authenticate("clients", cid, csecret))
			{
				context.Validated();
			}
		}
		return Task.FromResult<object>(null);
	}

	public override Task GrantResourceOwnerCredentials(OAuthGrantResourceOwnerCredentialsContext context)
	{
		var svc = context.OwinContext.Environment.GetUserAccountService<UserAccount>();
		UserAccount user;
		if (svc.Authenticate("users", context.UserName, context.Password, out user))
		{
			var claims = user.GetAllClaims();

			var id = new System.Security.Claims.ClaimsIdentity(claims, "MembershipReboot");
			context.Validated(id);
		}

		return base.GrantResourceOwnerCredentials(context);
	}
}
```


In the ValidateClientAuthentication method, I have code that validates the client credentials and calls `OAuthValidateClientAuthenticationContext.Validated()` method if the credentials matches a record in the database. If the client is validated, then it moves on to validate the user or resource owner credentials (username and password), and calls the Validated() method on the instance of `OAuthGrantResourceOwnerCredentialsContext` that will be passed in to the method GrantResourceOwnerCredentials. 

Now we need to wire up the authorization server middleware to the Owin pipeline. There is a shorthand extension method on IAppBuilder to use this middleware, which is UseOAuthAuthorizationServer. We will use this extension method to configure the OAuth 2.0 endpoints in the Startup class:

```csharp
public void Configuration(IAppBuilder app)
{
    SimpleInjectorWebApiInitializer.Initialize(app);

    app.UseOAuthAuthorizationServer(new Microsoft.Owin.Security.OAuth.OAuthAuthorizationServerOptions
    {
        AllowInsecureHttp = true,//should be disabled in production
        Provider = new MyOAuthAuthorizationServerProvider(),
        TokenEndpointPath = new PathString("/token")
    });
}
```

Also, we're going to add the bearer token authentication middleware for consuming the token:
```csharp
public void Configuration(IAppBuilder app)
{
    SimpleInjectorWebApiInitializer.Initialize(app);

    app.UseOAuthAuthorizationServer(new Microsoft.Owin.Security.OAuth.OAuthAuthorizationServerOptions
    {
        AllowInsecureHttp = true,
        Provider = new MyOAuthAuthorizationServerProvider(),
        TokenEndpointPath = new PathString("/token")
    });

    // token consumption
    var oauthConfig = new Microsoft.Owin.Security.OAuth.OAuthBearerAuthenticationOptions
    {
        AuthenticationMode = Microsoft.Owin.Security.AuthenticationMode.Active,
        AuthenticationType = "Bearer"
    };
    // Enable the application to use bearer tokens to authenticate users
    app.UseOAuthBearerAuthentication(oauthConfig);
}
```

To get a token, we need to make a post request to localhost/token passing in the necessary parameters. For example:
```
POST http://localhost:19923/token
Content-Type: Application/x-www-form-urlencoded
 
username=jbloggs&password=pass1234&grant_type=password
```

At this point, we'll obtain a result like the following:
```json
{
  "access_token": "XoHocsL0wOJBVVrjvSj6GpdGD-VrPD2ainZGyeZ8ji0aTq33epyHw72POhB8evYn41fzaSnjx7eo0iclADQBWTMgnghgZdXzNoLo6hwf4Y3SiB0aPTPgZi6PJwoGQK_aMWW62770jo6PBznPrSO0AOZUIrpxjUSZze90-HJjsM9ZgATIWdIvuiICqVjW7n5Z-o0GNSoDTIm-4k2zee0-c_lifHuLmW97IbsQ3I4gMz1SCBReSJtXJc8noPHvgwhFB_qZ2R1-TxR64nUVgxYYtBAoy9n6WKTgNAqrnUwsa0jfMk6wselrLwMGq-R-6_AX4bkh16OZBTGa5hXVWoLPIHl2JTKCkO2DsX2jqvp3J7PObRkZWMyUOyzwhnQWu_XTpn4ogwtcJvLulfiA6W01s8qiUQO--Xefm38ngu5HTM4",
  "token_type": "bearer",
  "expires_in": 1199
}
```

And that's all we need to get rolling. As you can see, the token has a type and an expiration date. Clean and secure! [I've got a sample project on here on GitHub](https://github.com/pmbanugo/MembershipRebootPasswordCredentialGrantSample/).