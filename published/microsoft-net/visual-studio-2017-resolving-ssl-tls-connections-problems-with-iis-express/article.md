Building ASP.NET web projects that enable [HTTPS](https://en.wikipedia.org/wiki/HTTPS) helps ensure website authenticity, integrity of the data exchanged between the client and the website, and protections over users' interactions with the website from prying internet service providers and third-party snoopers.

Using HTTPS for production websites relies on trusted [public key certificates](https://en.wikipedia.org/wiki/Public_key_certificate) that are issued by [certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority). These certificates are issued by specialist commercial enterprises or nonprofit organizations like [CACert.org](http://www.cacert.org/) and [Let's Encrypt](https://letsencrypt.org/).

Developers using Microsoft Visual Studio to build websites can create a local certificate to provide HTTPS functionality. Visual Studio includes [IIS Express](https://docs.microsoft.com/en-us/iis/extensions/introduction-to-iis-express/iis-express-overview), a "lightweight" version of Internet Information Server designed for the needs of developers.

In particular, IIS Express:

* does not require Administrator rights on the local machine, making it convenient for developers working in an enterprise environment where the user rights on their development workstation are restricted;

* it runs automatically when the web project is run from Visual Studio and shuts down when stopped from Visual Studio or the browser is closed;

* it supports edit and reload and other debugging and development features included with Visual Studio;

* it can be run from the command line without requiring Visual Studio to be running; and

* the full HTTP server log can be displayed in the command window in real time as IIS Express serves requests.

Note that IIS Express **10** is the current version and ships with Visual Studio 2017 (all editions, including Community). An [IIS Express 10 download](https://www.microsoft.com/en-us/download/details.aspx?id=48264) is also available separately. IIS Express documentation on [docs.microsoft.com](https://docs.microsoft.com/en-us/iis/extensions/introduction-to-iis-express/iis-express-overview) and [IIS.net](https://www.iis.net/learn/extensions/introduction-to-iis-express) dates from 2010 to 2013 and covers IIS Express versions 7.5 and 8.0. Experience indicates that much of the information still applies to version 10.

## Identifying the problem

When IIS Express is installed with Visual Studio, the installation process creates an IIS Express Development Certificate that serves as the HTTPS certificate for websites running on IIS Express on the local machine. This certificate has an *expiration date* that is five years from the date on which the certificate is created.

Using HTTPS to connect to a website running in IIS Express will fail when the IIS Express Development Certificate is improperly bound to the port or the certificate has expired or the certificate has been improperly installed.

Common browser error messages for HTTPS failures include the following:

    this site cannot provide a secure connection
    localhost sent an invalid response
    err_ssl_protocol_error
    This web page is not available ERR_CONNECTION_RESET

The specific error message you receive will depend on the browser version you are using for debugging.

## Before you begin

This guide provides one approach to resolving SSL/TLS connection problems experienced when running ASP.NET web projects using the IIS Express development web server. It is oriented to the current version of Visual Studio (as of the time of writing). It has not been tested for compatibility with prior versions of Visual Studio or IIS Express.

Most importantly, this guide is intended for developers who are using automatic port configuration for their web projects. Visual Studio assigns unique ports to projects for both HTTP and HTTPS on the local machine (localhost) and keeps this information in sync in various places. Unless you have a compelling reason to assign specific ports, life is more convenient if you leave this alone.

This guide is also based on the automatic creation of a self-signed certificate for use in development. If you have any reason to use a specific certificate, the instructions in this guide will not apply to you.

In addition, we have no way of knowing if your system is configured in a way that might adversely effect the results of following the procedures documented here or if these instructions are incomplete or inaccurate for a specific configuration.

### Prerequisites

* Visual Studio 2017 (version 15.3.4 or higher)
* IIS Express 10
* Google Chrome version 60+ (if Chrome is being used for debugging)

### Recommended preparation

Before getting started, you may want to do the following to ensure you can recover from any unforeseen consequences:

* Check in your code and publish it to a remote repository,

* [Create a current system restore point](https://support.microsoft.com/en-us/help/4027538/windows-create-a-system-restore-point),

* Create a current backup or sync your local data folders with your cloud drive(s).

 It's always good to have a current backup, and there are multiple services that make doing so easy and inexpensive.

## Resolving the problem

> PluralSight and the author disclaim all liability for the reliability, accuracy, and effect of following these instructions. Use at your own risk.

### Configuration prerequisites

As noted above, there are a few constraints on the use of these instructions:

* IIS Express will automatically assign port numbers,

* **localhost** is the server name for the IIS Express host (not the machine name), and

* although IIS Express can host or [php](https://eev.ee/blog/2012/04/09/php-a-fractal-of-bad-design/) projects, this guide does not address them.

#### Google Chrome version

Google introduced a key change in the way it handled SSL certificates in version 58. Working with Microsoft, they resolved differences in version 60. Many recent problem reports associated with SSL debugging with Visual Studio and Chrome come from this issue.

Before proceeding with this solution, check your Chrome version number and be sure it is 60 or higher:

**Menu** > **Help** > **About Google Chrome**
(**Alt** > **down arrow** > **e** > **g**)

#### SSL port assignment

IIS Express reserves a limited number of ports for automatic SSL binding. Be sure you are running (or attempting to run) SSL connections on ports in the following range:

    44300 to 44399 (inclusive)

If you are running an up-to-date version of Chrome (or Edge or Firefox or Opera) and you are using a valid SSL port number and you are still getting errors, proceed with the process presented in this guide.

### Resetting local certificates

The following instructions are based on the premise that you want your development environment to automate as many routine, repetitive tasks as possible. Accordingly, the instructions intend to generalize and automate Visual Studio and IIS Express.

These instructions are also based on the premise that a clean slate is often the best way to resolve a problem and prevent future problems. Accordingly, these instructions are intended to provide a *complete reset of your IIS Express Development Certificate* by removing any currently existing certificates and creating a new one that will bind successfully to the appropriate ports.

*[Author's note: If your development environment doesn't meet the prerequisites for this solution, or you have other constraints that prevent you from following these instructions, I hope you'll still be able to derive some value from the information presented below.]*

#### Removing local certificates

The first step in the repair process is to clear out existing certificates for **localhost**. You may want to [create a system restore point](https://support.microsoft.com/en-us/help/4027538/windows-create-a-system-restore-point) and a [recovery drive](https://support.microsoft.com/en-us/help/4026852/windows-create-a-recovery-drive) before beginning.

Open a command window (Run As Administrator).

Open Microsoft Management Console: type `MMC` at the command prompt and press **Enter**.

    
    ![command prompt](https://raw.githubusercontent.com/pluralsight/guides/master/images/2bfc8ec3-994a-4c84-aad1-da2780e190cc.png)

Add the **Certificates** snap-in.

    **File** > **Add or Remove Snap-ins** > **Certificates** > **Add**


![MMC Add or Remove Snap-ins](https://raw.githubusercontent.com/pluralsight/guides/master/images/a5a50d5d-68ff-4b10-9d73-8bf17b781c26.png)

When you click **Add** you will be presented with a dialog box to select the account:

![MMC Select Account](https://raw.githubusercontent.com/pluralsight/guides/master/images/39d0d316-94cd-4d8d-8e59-1165e37c5cfc.png)

Select **My User Account** and click **Finish**.

Under **Console Root / Certificates - Current User / Personal / Certificates** locate any certificates for which the value in the **Issued To** column is `localhost`. (Under most circumstances there shouldn't be any.) Select the certificate(s), right-click, and select **Delete**.

Under **Console Root / Certificates - Current User / Trusted Root Certification Authorities / Certificates** locate any certificates for which the value in the **Issued To** column is `localhost`. Select the certificate(s), right-click, and select **Delete**.

There will be a longer list of certificates. Some of the certificates issued to `localhost` may be current or expired. Delete all of them: the goal is to remove potential conflicts with the results of the IIS Express repair process described below.

![MMC Delete Personal Trusted Certificates](https://raw.githubusercontent.com/pluralsight/guides/master/images/b8b4eaa2-cbf1-4492-a91d-2f4495879fb1.png)

Close Microsoft Management Console.

Re-open Microsoft Management Console (from the command prompt in the command window you previously opened) and add the **Certificates** snap-in. This time, select **Computer account** at the **Certificates snap-in** dialog box and click **Finish**.

    You will be presented with a dialog box:

![MMC Select Computer](https://raw.githubusercontent.com/pluralsight/guides/master/images/0873f5a2-1d42-4c08-a9b2-7a70b2a20da5.png)

Select **Local Computer** and click **Finish**.

Under **Console Root / Certificates -  (Local Computer) / Personal / Certificates** locate any certificates for which the value in the **Issued To** column is `localhost`. Select the certificate(s), right-click, and select **Delete**.

![MMC Delete Local Computer Personal Certificates](https://raw.githubusercontent.com/pluralsight/guides/master/images/d081ae80-2099-421d-b8b5-ddf9267f60d4.png)

Under **Console Root / Certificates -  (Local Computer) / Trusted Root Certification Authorities / Certificates** locate any certificates for which the value in the **Issued To** column is `localhost`. Select the certificate(s), right-click, and select **Delete**.

Close Microsoft Management Console.

#### Creating a new certificate

This process uses the IIS Express installer to generate a certificate for `localhost` and get it installed in the right place, then it uses Visual Studio to bind the certificate to the ports used by IIS Express. These steps only need to be executed once to install the certificate. After installation, each subsequent web project that uses SSL will be assigned a SSL port bound to the certificate.

From the Windows desktop, open the old-style Control Panel:

    **Start** > **Windows System** folder > **Control Panel**

    *Note*: you must use the old-style Control Panel. Windows Settings / Apps will not give you the appropriate context menu option.

Select **Programs** > **Programs and Features** and find **IIS Express** in the list of programs.

Right-click on **ISS Express** and click **Repair**. Answer "Yes" to any confirmation dialog boxes that appear.


    ![Control Panel Programs and Features](https://raw.githubusercontent.com/pluralsight/guides/master/images/736d2c32-75fc-4a89-9f64-59e906a37d38.png)

The IIS Express installer will chug through the repair process and return you to the list of Programs and Features when it's finished.

Open Microsoft Management Console again and select the Certificates snap-in for the **Local Computer** account.

    Under  **Console Root / Certificates -  (Local Computer) / Personal / Certificates** you should see a new certificate Issued To `localhost` with a description of `IIS Express Development Certificate` and a Expiration Date 5 years minus 1 day from the current date.

Close Microsoft Management Console. (Leave the command window open, though. You're not done with it. )

Open Visual Studio and open the solution containing the web project you'd like to run in IIS Express with SSL.

Verify that the project is configured to use SSL by selecting the project in Visual Studio Solution Explorer and pressing **F4**. You should see a panel similar to the following:


![Visual Studio Development Server Properties](https://raw.githubusercontent.com/pluralsight/guides/master/images/101f0994-971a-4385-97f5-b72580af00e0.png)

Verify that **SSL Enabled** is set to `True`.

If you are working with a web project for which SSL has not yet been enabled, set **SSL Enabled** to `True`. Visual Studio will assign an unique SSL port number in the appropriate range (44300 - 44399) dynamically and set the **SSL URL** property.

Make note of the value for **SSL URL**.

Open the project properties by selecting the project in Solution Explorer and:

    pressing **Alt + Enter**, or

    right-clicking, and selecting **Properties...** from the context menu, or

    selecting **Project** > **_project name_ Properties** from the menu bar.

Select the **Web** tab and enter the value from **SSL URL** in the step above in the **Project URL** field:

![Visual Studio Project Properties](https://raw.githubusercontent.com/pluralsight/guides/master/images/71beec54-1fb1-4db0-b8b6-73592dc9dcfa.png)

Save your changes: **File** > **Save** (**Ctrl + S**).

You may be shown a dialog box like the following if you haven't run the web project using the port before:

![Visual Studio Virtual Directory confirmation](https://raw.githubusercontent.com/pluralsight/guides/master/images/9e3ceb1e-383a-493f-886d-59638a300368.png)

Select **Yes**.

Run the project in Visual Studio:

    **Debug** > **Start Debugging** (**F5**)

You will be presented with a dialog box:

![Visual Studio SSL configuration warning](https://raw.githubusercontent.com/pluralsight/guides/master/images/d45fd286-70a8-4269-973d-af50707f8091.png)

Select **Yes**.

This enables Visual Studio to trust the SSL certificate you created using the IIS Express Repair process, which is installed in:

**Certificates -  (Local Computer) / Personal / Certificates**

You will be presented with another dialog box:

![Visual Studio SSL Certificate Installation Warning](https://raw.githubusercontent.com/pluralsight/guides/master/images/09605b83-6ece-4383-921c-e418867cc5f5.png)

Select **Yes**.

This will take the certificate that you created and bound to the port selected in the Project Development Server Properties (F4) and the Project Properties (Alt + Enter) and install it in:

**Console Root / Certificates - Current User / Trusted Root Certification Authorities / Certificates**

The project should run (unless you have build or runtime problems otherwise) and appear in the browser like below:

![BlipTLS project in Chrome using SSL](https://raw.githubusercontent.com/pluralsight/guides/master/images/5c8af427-7849-43b4-b29c-36c49f0f6620.png)

Note the following characteristics:

* The green lock icon and "Secure" is visible to the left of the URL. (Different indicators are used in other browsers.)

* The protocol is indicated as `https`.

* The port being served is the automatically generated one for SSL connections.

Examine the values for running applications in IIS Express:

    Find the IIS Express Tray Icon, open the context menu (right-click), and select **Show All Applications**.

![IIS Express Tray Icon Context Menu](https://raw.githubusercontent.com/pluralsight/guides/master/images/32fafe77-42fe-44ad-abfc-7ee376b0e4ed.png)

This will display the IIS Express Running Applications dialog box:

![IIS Express Running Applications](https://raw.githubusercontent.com/pluralsight/guides/master/images/e08b31b4-ee51-4929-ab00-3736861863d5.png)

Note that IIS is running the website on both the http and https ports specified in the Project Development Server Properties.

Stop the project in Visual Studio. The browser window should close and IIS Express should stop running. (The tray icon will disappear.)

## Verifying certificate installation

As a final step, you can verify the installation of the new IIS Express Development Certificate.

Open Microsoft Management Console and add the **Certificates** snap-in. Select the **Current user** account.

Navigate the folder structure to:

    **Console Root / Certificates - Current User / Trusted Root Certification Authorities / Certificates**

    You should see that a `localhost` certificate has been installed:

![Microsoft Management Console Certificates Current User](https://raw.githubusercontent.com/pluralsight/guides/master/images/62c288c7-c08e-442c-a1f4-ac5e945abfaf.png)

Note that the `localhost` IIS Express Development Certificate created by the IIS Express Installer repair process now appears in this list with a special icon:


![Microsoft Management Console Detail](https://raw.githubusercontent.com/pluralsight/guides/master/images/7c477721-619b-441d-854f-b9c7e71d77f0.png)

## Conclusion

Although the process for repairing IIS Express SSL certificate binding problems with Visual Studio web projects is somewhat lengthy, it is a reliable and repeatable way to reset this functionality using built-in processes in the IIS Express installer and Visual Studio. This is important because installing the certificates and binding them correctly is not just a matter of copying the certificates to the right place in a folder structure.

SSL certificate port binding, an essential part of the process of configuring SSL for a web project, requires a number of precise steps these build-in processes handle behind the scenes. Using Visual Studio's automatic SSL port assignment eliminates the need to execute a number of steps to bind an SSL certificate to the SSL port for each new web project.

For best results, be sure you are using the latest versions of Visual Studio 2017 and IIS Express.

## More information

> PluralSight and the author disclaim all liability for the reliability, accuracy, and completeness of information presented by third-parties.

[docs.microsoft.com: Handling URL Binding Failures in IIS Express](https://docs.microsoft.com/en-us/iis/extensions/using-iis-express/handling-url-binding-failures-in-iis-express) - This article, from 2011, provides much useful information on using a custom SSL certificate and custom port bindings with IIS Express. It is of more help to those looking to customize their IIS Express configuration than those looking to resolve problems with their configuration.

---
*Written by A. J. Saulsberry*