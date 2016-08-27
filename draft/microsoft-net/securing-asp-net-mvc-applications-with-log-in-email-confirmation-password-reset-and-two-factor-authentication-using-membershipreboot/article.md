Security is often a discussed topic and having an easy way to manage users accounts and authentication is something we've always wanted. Thanks to the asp.net team, ways of doing this keep getting better everyday, and we can see, over the years, the libraries been released for this purpose. We have moved from ASP.NET Membership, to SimpleMembership, and now ASP.NET Identity. I can look back early on in my career and see myself struggling trying to extend or add extra configuration to the then ASP.NET Membership. With this evolution, we have also moved from Role based authorization to Claims based authorization. 

[MembershipReboot](https://github.com/brockallen/BrockAllen.MembershipReboot) is a Claims-based user account and identity management framework, that makes it easy for you to manage users accounts. It encapsulates important security logic while making it easy for developers to configure and extend. It's features include but not limited to: 

* Extensible templating for email notifications
* Single- or multi-tenant account management
* Account linking with external identity providers (enterprise or social)
* Notification system for account activity and updates
* Claims-aware user identities
* Flexible account storage design (you can choose to store your data in a relational or non-relational data store)

In this tutorial, I am going to show you how to secure your ASP.NET MVC apps using this library. I will show how to set it up, create a user account, enable account/email verification, two factor authentication, and allow password reset. I will be doing this with a new project, but for existing apps, you can start from the section that install the needed nuget packages.

# Let's Begin

## Create a new MVC Web Application

1. Select File > New Project > ASP.NET Web Application. Enter a name and click `OK`. 
2. Select the MVC template.
3. Click on the `Change Authentication`button and choose **No Authentication** and then click __OK__
4. Now we have the project set-up and we're ready to roll :+1 

## Install the nuget packages
Next on is to install the following nuget packages:
* > __Install-Package BrockAllen.MembershipReboot.Owin__

This is the OWIN implementation package that performs the job of issuing cookies to track the logged in user, performing two-factor authentication, and the ApplicationInformation details. It has dependencies on `BrockAllen.MembershipReboot`(which is the core of the framework), and `Microsoft.Owin.Security.Cookies`, therefore, these packages where installed when we ran the above command.

 * > __Install-Package BrockAllen.MembershipReboot.Ef __

This is the Entity Framework persistence implementation, responsible for persisting data to a SQL datastore. This is because it has a flexible storage design, where you can choose to store your data in either SQL or NoSQL database, and for this tutorial, I've chosen to work with a SQL datastore using this package.
