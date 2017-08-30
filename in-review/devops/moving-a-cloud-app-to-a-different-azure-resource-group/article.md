I used the new Visual Studio Team Services builds to deploy to an Azure CloudService web app. It created the web app automatically for me, but was put into a default resource group. I wanted to move it to the resource group which I had defined for other parts of the system. I found out that it can’t be done through the Azure Portal.

I’m new to using PowerShell to manage Azure and had to go through several blogs and steps to move my resource. It is hoped that this tutorial will save you all that work/time.

Steps in review
===============
- Install Web Platform installer
- Use WPI to install Azure PowerShell
- install
- run Azure PowerShell as Administrator
- change the execution policy in PowerShell
 - Set-ExecutionPolicy Unrestricted
- Install PackageManagement installer to be able to run Install-Module (https://www.microsoft.com/en-us/download/details.aspx?id=49186)  in PowerShell
 - We need to get AzureRm and AzureRm.Resources from the PowerShell Gallery
- Install-Module -Name AzureRM to install AzureRm from the powershell gallery
- Install-Module –Name AzureRM.Resources
- Login-AzureRmAccount
 - (web browser popup came up asking for username and password for Azure) ** This a command to remember!
- Get-AzureRmResourceGroup
 - find the ResourceId of your Web app (I didn’t figure out how to filter by name)
- Move-AzureRmResource -ResourceId "/subscriptions/{your subscription here}/resourceGroups/Default-Web-NorthCentralUS/providers/Microsoft.Web/sites/cdd-Validate" -DestinationResourceGroupName continuousDeliveryDemo
 - There wasn’t a –ResourceName option, that would have made it easier 

/* TODO: Add more step by step process (with download/further info links) for the above steps.  

Here are some other articles that were useful to me.

https://www.powershellgallery.com/GettingStarted?section=Get%20Started

https://azure.microsoft.com/en-us/blog/azps-1-0/

https://channel9.msdn.com/Series/Windows-Azure-Virtual-Machines-and-Networking-Tutorials/Introduction-to-Windows-Azure-PowerShell – good 9 minute introduction video. 

Hint: Run $PSVersionTable.PSVersion in PowerShell to get the version of PS you have.

See [my blog](http://geekswithblogs.net/Aligned/) for more of my articles.
