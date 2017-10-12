# How to pull a list of users from Office 365 into a CSV

When using Microsoft Office 365, we sometimes find problems in the regular user interface. Usually problems occur either because the feature set we are looking for does not exist or because we need an automated way of doing something that 365 only allows us to do manually.

In these situations, PowerShell is the tool of choice as it allows for remote administration for all components within Office 365. In this post, we will walk through the steps needed to work with Office 365 using PowerShell.

## Office 365 PowerShell Setup for remote Administration
Getting your system configured and ready is the first step. You will first need to download the following components: (links below respective picture)

1. Microsoft Online Service Sign-in Assistant for IT Professionals 
2. Windows Azure Active Directory Module for Windows PowerShell
3. SharePoint Online Management Shell

Install each component in the order listed ensuring that each one completes successfully. Make sure to use the correct version based on your operating system (32 bit or 64 bit).

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/4923efc3-8217-4f02-9a8f-fcdbe5f193f5.png "Wizard Image One")
<center>(https://www.microsoft.com/en-us/download/details.aspx?id=28177)</center>

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/681fa419-84f2-4cb8-b73a-9e970d4cece1.png "Wizard Image Two")
<center>(https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-adv2?view=azureadps-2.0#installing-the-azure-ad-module)</center>

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/0be4b2c7-a8be-40c9-a955-6fb65d711430.png "Wizard Image Three")
<center>(https://www.microsoft.com/en-us/download/details.aspx?id=35588)</center>

Now launch a standard (non-admin) PowerShell window and run the following command:

```powershell 
Set-ExecutionPolicy RemoteSigned 
```

This will ensure that all PowerShell scripts that you run are not blocked by the security measures of PowerShell.

You are now setup as an administrator and ready to use Office 365 PowerShell commands. As a note, you will notice that there is not an installation for Exchange Online PowerShell cmdlets. Connecting to Exchange Online is completed using the New-PSSession and Import-PSSession commands instead.

## Connect to Office 365 using PowerShell
To run the needed PowerShell commands, for ease and convenience you will use the PowerShell ISE tool found within Windows. To launch this, if you are using Windows 8 or higher, then you can simply press the start menu and type “**PowerShell ISE**” from the **run cmd/search bar** and launch it from there. Once it is found, right click and choose “**Run as administrator**”.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/9095f91-e635-4c48-894e-8251657e2860.png "")

Once the PowerShell ISE loads, you can then run commands within the top half, and see the execution and results in the bottom half. If the top section is not visible, click the “**Script**” link and it will split the screen into two sections.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/ca751d7a-1913-4f2e-a30d-0da2401c147c.png "")

In the script pane, you can now initiate a connection to Office 365 by typing the following PowerShell commands in order. The first command will allow you to set a variable to your Office 365 credentials.

```powershell
$credential = get-credential
```

When prompted type your username and password within the popup window, once you have typed them, press OK, which will close the dialog and populate the variable.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/53acc76d-3e0e-4d42-883b-a9c172cd5021.png "")

To validate the variable, in the blue section at the bottom of the PowerShell ISE, simply type $credential, then press enter, and it will display the populated value.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/3e8d6301-2969-4b16-87a6-860cbf540f7a.png "")

Now that you have the credentials stored and available, you need to import the Office 365 PowerShell Modules directly into the PowerShell ISE window. To do this, type the following:

```powershell
Import-Module MSOnline
```

Once this is complete, you can initiate a connection to Office 365 with the stored credential object using the following command.

```powershell
Connect-MsolService -Credential $credential
```

Barring any errors, you should now have a connection to Azure Active Directory. To connect to SharePoint Online you will need to run the following command, which imports the SharePoint Online PowerShell cmdlets.

```powershell
Import-Module Microsoft.Online.Sharepoint.PowerShell
```

Once this has completed you will have access to all the modules and cmdlets within your current PowerShell session. Now you can create a connection to SharePoint Online by passing in the [URL of the SharePoint Online Administration Center site](https://technet.microsoft.com/en-us/library/fp161392.aspx)

```powershell
Connect-SPOService -url https://{tenant}-admin.sharepoint.com -Credential $credential
```

To administer Skype for Business and Exchange Online, you will need to install the Skype connector and as mentioned before use a different connection approach for Exchange Online.

## Iterate users in Office 365 using PowerShell
Now that you have a connection to Office 365, you can run some other commands to interrogate services or user information.

Office 365 PowerShell contains a command called Get-MsolUser which can be used to iterate all users within the Office 365 tenant. Within the PowerShell ISE window type the following, which will retrieve all users and load them into a grid for easy viewing.

```powershell
Get-MsolUser | Out-GridView
```

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/716801ca-10f7-432c-a0ff-8af883dd5718.png  "")

Close the Grid view windows and modify the command to now be the following (showing membership status):

```powershell
Get-MsolUser | Get-Member | Out-GridView
```

When this runs instead of getting a list of users, you will now be presented with the list of properties that you can use when modifying user accounts or even just iterating accounts to check values.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/210c3c26-6fed-4e75-85da-8e182e3afe30.png  "")

The same command can actually be modified again to only return a single user if needed.

```powershell
Get-MsolUser -UserPrincipalName "user@domain.com" | Out-GridView
```

This will once again output using the standard grid popup window. There are other properties that can be used to filter the list of user accounts that you want to be returned. You could retrieve users using some of the examples below.

**By Department** 
```powershell 
Get-MsolUser -Department "Human Resources" | Out-GridView 
```

**Enabled Accounts Only**
```powershell
Get-MsolUser -EnabledFilter EnabledOnly | Out-GridView
```

**By Search String**
```powershell
Get-MsolUser -SearchString "Steve" | Out-GridView
```

As you can see iterating user accounts is simple and easy using the standard PowerShell Get-MsolUser cmdlet, as well as filtering the users accounts you want to see.


## Export users from Office 365 using PowerShell

Exporting users from Office 365 into a flat file, still relies on the same commands you used previously for getting a user, so namely Get-MsolUser. In the previous examples, you simply loaded the Grid View, though that works well, it does not give you the option to perform extra work on the raw data.

Exporting to a CSV file within PowerShell is really as simple as using the Export-Csv PowerShell cmdlet. To export all users from Office 365 to a CSV file, which will contain the UPN, Display Name, Country and Department of the user, use the following command.

```powershell
Get-MsolUser | Select-Object UserPrincipalName, DisplayName, Country, Department | Export-Csv c:\Export\Export-All-Users.csv
```

This command will iterate all users and generate the CSV file.

![alt text](https://github.com/helloitsliam/assets/blob/master/images/pl-1/2eceb267-ce21-4208-9fc9-7b815bba362e.png  "")

This command could be modified to include any of the filters such as Department or if they are licensed accounts. This is done by modifying the PowerShell command to include either a “Where” clause or pass the property within the Get-MsolUser command.

```powershell
Get-MsolUser Property: Get-MsolUser -EnabledFilter EnabledOnly | Select-Object UserPrincipalName, DisplayName, Country, Department | Export-Csv c:\Export\Property-Filter-Export-Enabled-Users.csv
```

```powershell
Get-MsolUser with Where Clause: Get-MsolUser | Where-Object { $_.isLicensed -eq “TRUE” } | Select-Object UserPrincipalName, DisplayName, Country, Department | Export-Csv c:\Export\Where-Clause-Export-Enabled-Users.csv
```

Using the standard Get-MsolUser with its properties, any combination of queries can be run. With this command combined with the Export-Csv command, all users can easily be exported to a list as needed.

Hopefully this article has helped you work more fluidly with the Office 365 environment. Understanding how to use PowerShell in conjunction with Office gives the user far more control and enables Office to handle many important tasks that would have been difficult to accomplish otherwise.

Thanks for reading! Please leave all comments and feedback in the discussion section below!
