Security is often a discussed topic and having an easy way to manage users accounts and authentication is something we've always wanted. Thanks to the asp.net team, ways of doing this keep getting better everyday, and we can see, over the years, the libraries been released for this purpose. We have moved from ASP.NET Membership, to SimpleMembership, and now ASP.NET Identity. I can look back early on in my career and see myself struggling trying to extend or add extra configuration to the then ASP.NET Membership. With this evolution, we have also moved from Role based authorization to Claims based authorization. 

[MembershipReboot](https://github.com/brockallen/BrockAllen.MembershipReboot) is a user account management library, that makes it easy for you to manage users accounts. It encapsulates important security logic while making it easy for developers to configure and extend. It's features include but not limited to: 

* Extensible templating for email notifications
* Single- or multi-tenant account management
* Account linking with external identity providers (enterprise or social)
* Notification system for account activity and updates
* Claims-aware user identities
* Flexible account storage design (you can choose to store your data in a relational or non-relational data store)

In this tutorial, I am going to walk you through, adding this library to