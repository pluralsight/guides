![](http://i.imgur.com/gFTi3A2.png)

# How to publish our calculator application to the Apple's App Store and Google's Play Store

This is the fourth post in a series of posts which will teach you how to take advantage of your web development knowledge in building hybrid applications for iOS and Android. The series comprises of the following published tutorials:

+ [How to get started with Ionic framework on Windows and Mac](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows)
+ [How to create a calculator application with Ionic framework by using Ionic Creator for UI](http://tutorials.pluralsight.com/review/how-to-create-a-calculator-application-with-ionic-framework-by-using-ionic-creator-for-ui/article.md)
+ [How to polish, create icons and splash screen images, add ads, share and test our calculator application](http://tutorials.pluralsight.com/review/how-to-polish-create-icons-and-splash-screen-images-add-ads-share-and-test-our-calculator-application/article.md).

> This post wraps the series in a whole where we showed how to create an application starting with an idea, through creating a prototype and implementing the application using Ionic framework to finally (in this tutorial) publishing it in the stores.

> In all the future tutorials, we'll finish the app (there will be some more practical app examples like taking a picture with your phone and uploading it to a server, creating a note taking application, etc.) and for actual deployment we'll just note that you can take a look at this 4th tutorial which covers the exacts steps you need to make in order to publish a finished app to the Apple's App Store and Google's Play Store.

You can see the finished product on the header image and you can check out the application by downloading it from the App Store or Play Store [here](http://nikola-breznjak.com/apps/SuperSimpleCalculator/).

If you've been following this tutorial series then you probably already have the Calculator application ready and running on your machine in the web browser by using the `ionic serve` command. If you just dropped by, or you would like to "start anew" you can clone the finished version of the 3rd tutorial from [Github](https://github.com/Hitman666/Ionic_3rdTutorial). After cloning, you should run `npm install` to install all the dependencies. After this the command `ionic serve` should run the application locally on your computer, and you should see it open up in your default browser.

This fourth post is divided into two sections, each for one store. We'll start with Google's Play Store.
    
## Google Play Store

In this section we're going to go over the following steps:

+ How to prepare the application for production
+ How to create the keystore file
+ How to build the production ready application
+ How to get the developer license
+ How to sign the APK file
+ How to optimize the APK file
+ How to build the update of your application
+ How to create the application listing
+ How to upload the application to the Play Store

### Preparing our application for production
Basically, you should remove any unnecessary plugins or libraries which you may have installed during development that you didn't end up using. In our example, we can safely remove the `cordova-plugin-console` by executing the following command: `cordova plugins remove cordova-plugin-console`.

Next, in your `package.json` file you should change the `name` and `description` properties. This is how I've set up mine:

```
{
  "name": "SuperSimpleCalculator",
  "version": "1.0.0",
  "description": "SuperSimple Calculator by Nikola Brežnjak",
  "dependencies": {
    "gulp": "^3.5.6",
    "gulp-sass": "^1.3.3",
    "gulp-concat": "^2.2.0",
    "gulp-minify-css": "^0.3.0",
    "gulp-rename": "^1.2.0"
  },
  "devDependencies": {
    "bower": "^1.3.3",
    "gulp-util": "^2.2.14",
    "shelljs": "^0.3.0"
  },
  "cordovaPlugins": [
    "cordova-plugin-device",
    "cordova-plugin-whitelist",
    "cordova-plugin-splashscreen",
    "com.ionic.keyboard",
    "cordova-plugin-admobpro"
  ],
  "cordovaPlatforms": [
    "ios",
    "android"
  ]
}
```

In the `config.xml` file make sure that you change the `widget id`, `name`, `description` and `author` elements. My file looks like this now:

```
<widget id="com.nikola-breznjak.calculator" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0" xmlns:gap="http://phonegap.com/ns/1.0">
  <name>SuperSimple Calculator</name>
  <description>
        An Ionic Framework SuperSimple Calculator application.
    </description>
  <author email="nikola.breznjak@gmail.com" href="http://www.nikola-breznjak.com/">
      Nikola Brežnjak
  </author>
```

### Setting up a keystore
A keystore file stores the security key that we'll use later to sign our application. By doing this we can verify that we're the author of the application. You can learn more about application signing from the [official documentation](http://developer.android.com/tools/publishing/app-signing.html).

In order to create a keystore, execute the following command (change *SuperSimpleCalculator* keyword with your own):

`keytool -genkey -v -keystore SuperSimpleCalculator.keystore -alias SuperSimpleCalculator -keyalg RSA -keysize 2048 -validity 10000`

You will be asked for a password two times (do not forget it!). The process looks like this:

```
C:\Users\Nikola\Desktop\IonicTesting\Ionic_4thTutorial>keytool -genkey -v -keystore SuperSimpleCalculator.keystore -alia
s SuperSimpleCalculator -keyalg RSA -keysize 2048 -validity 10000
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  Nikola Brežnjak
What is the name of your organizational unit?
  [Unknown]:
What is the name of your organization?
  [Unknown]:  Nikola Brežnjak
What is the name of your City or Locality?
  [Unknown]:  Zagreb
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:  HR
Is CN=Nikola Bre×njak, OU=Unknown, O=Nikola Bre×njak, L=Zagreb, ST=Unknown, C=HR correct?
  [no]:  What is your first and last name?
  [Nikola Bre×njak]:
C:\Users\Nikola\Desktop\IonicTesting\Ionic_4thTutorial>keytool -genkey -v -keystore SuperSimpleCalculator.keystore -alia
s SuperSimpleCalculator -keyalg RSA -keysize 2048 -validity 10000
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  Nikola Breznjak
What is the name of your organizational unit?
  [Unknown]:
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:  Zagreb
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:  HR
Is CN=Nikola Breznjak, OU=Unknown, O=Unknown, L=Zagreb, ST=Unknown, C=HR correct?
  [no]:  yes

Generating 2.048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 10.000 days
        for: CN=Nikola Breznjak, OU=Unknown, O=Unknown, L=Zagreb, ST=Unknown, C=HR
Enter key password for <SuperSimpleCalculator>
        (RETURN if same as keystore password):
[Storing SuperSimpleCalculator.keystore]
```

This command will create a `SuperSimpleCalculator.keystore` file. You should place this file in a place where you'll remember it, because you'll need this file for any app updates you'll want to push to the Play Store. However, just to make things clear; **for every new application that you make, you need to create a new keystore file**.

### Building the production ready application 
To create a production version of your application, execute the following command:

`cordova build --release android`

This will generate a new **unsigned** APK file (Android app file type, just like .exe is on Windows), inside the `platforms/android/ant-build/` folder. The build process should end with an output similar to this:

```
BUILD SUCCESSFUL
Total time: 35.208 secs
Built the following apk(s):
    C:\Users\Nikola\Desktop\IonicTesting\Ionic_4thTutorial\platforms\android\build\outputs\apk\android-release-unsigned.apk
```

Here you can see the exact full path to the unsigned, production (release-ready) APK file of your application.

### Signing the APK file
Now we'll sign the unsigned version of the APK with a keystore. We can do this with the tool called `jarsigner`. First, copy the keystore to the same directory where your unsigned APK file is (in my case: `C:\Users\Nikola\Desktop\IonicTesting\Ionic_4thTutorial\platforms\android\build\outputs\apk\`). Next, rename the .apk file to something more meaningful, like for example SuperSimpleCalculator-release-unsigned.apk.

In your terminal navigate to this folder and execute the following command:

`jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore SuperSimpleCalculator.keystore SuperSimpleCalculator-release-unsigned.apk SuperSimpleCalculator`

You will be prompted for the keystore password. You can test that the app is signed properly with the following command:

`$ jarsigner -verify -verbose -certs CordovaApp-release-unsigned.apk`

This should return something like `jar verified`.

If you get a warning like this:
```
Warning:
No -tsa or -tsacert is provided and this jar is not timestamped. Without a timestamp, users may not be able to validate this jar after the signer certificate's expiration date (2043-04-07) or after any future revocation date.
```
you don't have to worry (you can learn more about it [here](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/jarsigner.html#Options)), and you can move safely on.

### Optimizing the APK file
With the `zipalign` tool we can optimize the APK file so that it reduces the amount of space and RAM required by the app on a device. You can learn more about zipalign from the [official documentation](http://android-developers.blogspot.hr/2009/09/zipalign-easy-optimization.html).

The zipalign tool just takes the name of the signed file (we still have *unsigned* in the name of our file because we didn't change it in the previous step) and the name of the file to generate. To optimize the APK file execute the following command:

`zipalign -v 4 SuperSimpleCalculator-release-unsigned.apk SuperSimpleCalculator.apk`

The process finishes with the `Verification succesful` output and you will get a **SuperSimpleCalculator.apk** file which you can submit to the Google Play Store.

You may get an error like:

> zipalign is not recognized as an internal or external command, operable program or batch file.

In this case, you need to search your computer for the zipalign file, and make sure it's in your system PATH to fix this problem. Alternatively, you can enter a full path to the zipalign file (in my case, on Windows, the the path to the file was `C:\Android\android-sdk\build-tools\21.1.2\zipalign.exe`).

### Building the update of your application
To build an update of an existing app you have to follow the same steps except that you don't need to create another keystore. You have to use the **same keystore** to sign the application for every update, becase the update will be otherwise rejected and you'll have to create a new app listing.

You must update the **version** in the **config.xml** file for the next release, otherwise the app will not be properly updated on your users' devices. To learn more about versioning you can take a look at the [official documentation](https://cordova.apache.org/docs/en/dev/config_ref/index.html), or this [StackOverflow question](http://stackoverflow.com/questions/25858547/phonegap-how-to-set-build-and-version-property-in-config-xml-for-ios).

### Creating the app listing and uploading the app to the Play Store
The first thing you should do is sign up for the Google developer program at [https://play.google.com/apps/publish/signup/](https://play.google.com/apps/publish/signup/). If you're not going to make applications as an individial then it's recommended that you create a separate Google account for your apps (you need it to sign up). You'll need to pay **$25 one-time fee** for a developer license.

You start by clicking the `Add new application` button on the right hand side as shown on the image below:

![](http://i.imgur.com/iabOLRX.jpg)

Now the popup appears, as show on the image below, and you have to put a `Title` for your application. Here you basically have options to `Upload the APK` and to `Prepare the Store Listing`.

![](http://i.imgur.com/TDtdXX0.jpg)

We're going to upload our APK to production, by simply clicking on the `Upload your first APK to Production` button once on the PRODUCTION tab, then selecting the signed APK that we made in the previous section.

![](http://i.imgur.com/O5WYiL3.jpg)

Once the upload is finished you'll see something like it's shown on the image below, and on the left hand side you'll see the green checkmark next to the APK, meaning that you've completed the APK upload step.

![](http://i.imgur.com/f8Qfl68.jpg)

Next you have to set the graphics for your application. Please note that you have to put at least 2 screenshots!

![](http://i.imgur.com/E440B1O.jpg)

Then we have to select a category:

![](http://i.imgur.com/3gcXBmh.jpg)

Next up is the Content rating where you'll be asked quite a few questions (here you'll be able to better choose the category of your application because it will be listing a few example applications):

![](http://i.imgur.com/62AV13D.jpg)

Next you have to click on the `Save questionnaire`, and after that on the `Get rating`  button. After the content rating is done, you'll be able to click the `Apply rating` button.

Finally, you will now have an option to click the `Publish app` button:

![](http://i.imgur.com/KfMoTnb.jpg)

With this your app now goes into the review process which in my case took 4 hours. Until the app is published you will see a `In review` status, and once the app is published you'll see the following:

![](http://i.imgur.com/NaQmZ4g.jpg)

If you click on the [View in play store](https://play.google.com/store/apps/details?id=com.nikola_breznjak.calculator) link you'll see the listing in Play Store:

![](http://i.imgur.com/XKuR9nh.jpg)

You can use this link to share your application on a simple landing page like [this one](http://nikola-breznjak.com/apps/SuperSimpleCalculator/):

![](http://i.imgur.com/FMdbNhX.jpg)

## Apple App Store
First, you need to enroll in [Apple Developer Program](https://developer.apple.com/programs/). As with Google, if you have a personal account with Apple, you can create an additional one for your applications.

As I mentioned in the first tutorial, the fee is 99$ paid yearly and in my case the process took 2 days to be accepted in the program. Also, you'll have to have a Mac computer. Sure, there are ways around it, but I don't recommend them. For starters, a cheap Mac Mini will do just fine.

### Connecting Xcode with your developer account
After you receive your developer status, open Xcode on your Mac and go to `Preferences -> Accounts` and add your account to Xcode by clicking the `+` button on the lower left hand side, and follow the instructions:

![](http://i.imgur.com/ByEbzsJ.png)

### Signing
Now that you linked Xcode with your developer account, go to `Preferences -> Accounts`, select your Apple Id on the left hand side and then click the `View Details` button shown on the previous image. You should see the popup similar to the one on the image below:

![](http://i.imgur.com/zTEqbqg.png)

Click the `Create` button next to the `iOS Distribution` option.

You can learn more about maintaining your signing identities and certificates from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html).

### Setting up the app identifier
Next, through the [Apple Developer Member Center](https://developer.apple.com/membercenter) we'll set up the app ID identifier details. Identifiers are used to allow an app to have access to certain app services like for example Apple Pay. You can login to Apple Developer Member Center with your Apple ID and password.

Once you're logged in you should choose `Certificates, Identifiers, and Profiles` option as shown on the image below:

![](http://i.imgur.com/1aiVMqG.png)

On the next screen, shown on the image below, select the `Identifiers` option under the iOS Apps.

![](http://i.imgur.com/tmRoIlg.png)

One the next screen, shown on the image below, select the plus (+) button in order to add a new iOS App ID.

![](http://i.imgur.com/BbK4KHq.png)

On the next screen, shown partialy on the image below, you'll have to set the name of your app, and use the `Explicit App ID` option and set the `Bundle ID` to the value of the `id` in your Cordova `config.xml` <widget> tag.

![](http://i.imgur.com/fsENfcP.png)

Additionally, you'll have to choose any of the services that need to be enabled. For example, if you use Apple Pay or Wallet in your app, you need to choose those option. By default you can't deselect `Game Center` and `In-App Purchase` options, but for details on how to do this you can take a look at [this Stackoverflow question](http://stackoverflow.com/questions/15913115/ios-how-can-i-deselect-game-center-and-in-app-purchase-when-i-try-to-register-my).

You can learn more about registering app identifiers from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html).

### Creating the app listing
Apple uses [iTunes Connect](https://itunesconnect.apple.com) to manage app submissions. After you login, you should see a screen similar to the one on the image below:

![](http://i.imgur.com/mvnrafK.png)

Here you have to select the My Apps button, and on the next screen select the + button, just below the `iTunes Conenct My Apps` header, as shown on the image below:

![](http://i.imgur.com/68rK26r.png)

This will show three options in a dropdown, and you should select the `New App`. After this the popup appears, as shown on the image below, where you have to choose the name of the application, platform, primary language, bundle ID and SKU. As you can see on the image below I had to name my application differently, because `SuperSimple Calculator` was taken by some other application already published in the store.

![](http://i.imgur.com/fs6EDva.png)

Once you're done, click on the `Create` button and you'll be presented with the following screen where you'll have to set some basic options like Privacy Policy URL, category and sub category.

![](http://i.imgur.com/K0dIeL4.png)

Now, before we fill out everything in the listing, we'll build our app and get it uploaded with Xcode. Then you'll come back to finish the listing.

You can learn more about managing your app in iTunes Connect from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/UsingiTunesConnect/UsingiTunesConnect.html).

### Building the app for production
In the root directory of your application execute the following command:
`cordova build ios --release`

If everything went well you'll see the **BUILD SUCCEEDED** output in the console.

### Oppening the project in Xcode
Now, open the `platforms/ios/SuperSimpleCalculator.xcodeproj` file in Xcode (of course you would change `SuperSimpleCalculator` with your own name).

![](http://i.imgur.com/3lSsexm.png)

Once the Xcode opens up the project, you should see the details about your app in the general view, as shown on the image below:

![](http://i.imgur.com/qHttt0n.png)

You should just check that the bundle identifier is set up correctly, so that it's the same as the value you specified earlier in the app ID. Also, make sure that the version and build numbers are correct. Team option should be set to your Apple developer account. Under the deployment target you can choose which devices your application will support.

Since we're on this setting screen right now, I should mention that it's **very important** that you select the **Requires full screen** option, otherwise you may (as I did) get the following error when uploading the app to the App Store (covered futher in the tutorial):
`Invalid Bundle. iPad Multitasking support requires these orientations: 'UIInterfaceOrientationPortrait,UIInterfaceOrientationPortraitUpsideDown,UIInterfaceOrientationLandscapeLeft,UIInterfaceOrientationLandscapeRight'. Found 'UIInterfaceOrientationPortrait,UIInterfaceOrientationPortraitUpsideDown' in bundle 'com.bitscoffee.PhotoMarks.iOS'.`

You can learn more about this error from [this StackOverflow question](http://stackoverflow.com/questions/32559724/ipad-multitasking-support-requires-these-orientations).

### Creating an archive of the application
In Xcode, select `Product -> Scheme -> Edit Scheme` to open the scheme editor. Next, select the `Archive` from the list on the left hand side. Make sure that the `Build configuration` is set to `Release` as shown on the image below:

![](http://i.imgur.com/WGZUw7t.png)

To create an archive choose a `Generic iOS Device`, or your device if it's connected to your Mac (you can't create an archive if simulator is selected), from the Scheme toolbar menu in the project editor, as shown on the image below:

![](http://i.imgur.com/4y3hs6p.png)

Next, select `Product -> Archive`, and the Archive organizer appears and displays the new archive. At this point Xcode runs some preliminary validations on the archive and it may display validation warnings in the project editor. At this point I got a few errors shown on the image below:

![](http://i.imgur.com/To14nx0.png)

This problem was resolved by the solution posted in this [Stack Overflow question](http://stackoverflow.com/questions/12184767/phonegap-cdvviewcontroller-h-file-not-found-when-archiving-for-ios) where basically the solution was to double click the `<multiple values>` under Header Search Paths` and change `$(OBJROOT)/UninstalledProducts/include` to `$(OBJROOT)/UninstalledProducts/$(PLATFORM_NAME)/include` as shown on the image below:

![](http://i.imgur.com/7LBPSTN.png)

Another error that I got was about bitcode support. You can learn more about the error and solution from this [StackOverflow question](http://stackoverflow.com/questions/30848208/new-warnings-in-ios9), but basically you have to set the Bitcode support to `No` in the `Build Options`, as shown on the image below:

![](http://i.imgur.com/38v8fhV.png)

After you fix any errors you can repeat the `Product -> Archive` step and when it succeeds it will bring up the window as shown on the image below:

![](http://i.imgur.com/wRbjZbr.png)

Here it's recommended that you click the `Validate` button which will check your app for any additional issues. If everything goes well, you'll see the following notification:

![](http://i.imgur.com/7NRtPF4.png)

At this point you can click the `Upload to App Store...` button, and if everything goes fine you'll have an uploaded app, and the only thing that's left to do is to complete the iTunes Connect listing and submit it for review!

If you get an email from iTunes Connect shortly after you uploaded the archive with the content similar to this:

```
 We have discovered one or more issues with your recent delivery for "Super Simplistic Calculator". Your delivery was successful, but you may wish to correct the following issues in your next delivery:

Missing Push Notification Entitlement - Your app appears to include API used to register with the Apple Push Notification service, but the app signature's entitlements do not include the "aps-environment" entitlement.
```

If you're not using Push Notifications then you don't have to worry, as stated in the [official Ionic framework forum](http://forum.ionicframework.com/t/missing-push-notification-entitlement/5436/20).

### Finishing the app list process
Now you should head back to the [iTunes Connect portal](https://itunesconnect.apple.com) and login. Next, click on the `Pricing and Availability` on the left hand side under `APP STORE INFORMATION. Basically, all that I've set here is the price 0 (Free) as you can see on the image below:

![](http://i.imgur.com/qEDWaZG.png)

> You don't have to worry about forgetting to insert any crucial and required information about your application, since you'll be notified about what's missing and what needs to be added/changed if you try to submit the app for review before all details are filled in.

Next, click on the `1.0 Prepare for Submission` button on the left hand side, as shown on the image below. When we uploaded our archive, iTunes Connect automatically determined which device sizes are supported. As you can also see on the image below, you'll need to upload at least one screenshot image for each of the various app sizes that were detected by iTunes Connect.

![](http://i.imgur.com/Rat0FRr.png)

From my personal experience, the easiest way to generate these images is to simulate the app in Simulator and make a screenshot of different screen sizes. However, please note that you have to make sure that your `Window -> Scale` property in the Simulator is set to `100%`, otherwise the image size will not be the one as expected in iTunes Connect.

Next you'll have to insert Description, Keywords, Support URL and Marketing URL (optionally), as shown on the image below:

![](http://i.imgur.com/46YV8Vw.png)

In the `Build` section you have to click on the `+` button and select the build which we've uploaded through Xcode in the previous steps, as shown on the image below:

![](http://i.imgur.com/RmZrrqX.png)

Next you'll have to upload the icon, edit the rating, and set some additional info like copyright and your information. Note that the size of the icon that you'll have to upload here will have to be 1024 by 1024 pixels. Thankfully, you can use the splash.png from the second tutorial. If you're the sole developer then the data in the `App Review Information` should be your own. Finally, as the last option, you can leave the default checked option that once your app is approved that it is automatically released to the App Store.

Now that we're finished with adding all of the details to the app listing, we can press `Save` and then `Submit for Review`. Finally, you'll be presented with the last form that you'll have to fill out:

![](http://i.imgur.com/BXbPvhf.png)

The only "tricky" question may be the one about Content rights. In our example we can safely select `No`. Since we're using AdMob in our application we have to select `Yes` in the `Advertising Identifier` and check the `Serve advertisements within the app` checkbox. You can learn more about it from this [StackOverflow question](http://stackoverflow.com/questions/31261926/ios-does-your-app-contain-display-or-access-third-party-content-admob).

After you submit your app for review you'll see the status of it in the My Apps as `Waiting for review`, as shown on the image below. Also, shortly after you submit your app for review you'll get a confirmation email from iTunes Connect that your app is in review.

![](http://i.imgur.com/Bm3LbQA.jpg)

Apple prides itself with a manual review process, which basically means it can take several days for your app to be reviewed. You'll be notified of any issues or updates to your app status. From my personal experience it usualy takes up to 5 days for the app to be reviewed. 

### Updating the app
Since you'll probably want to update your app at some point you'll first have to update the build and version numbers in the Cordova `config.xml` file and then rebuild the application and open it up from the Xcode and follow the same steps all over again.

Once you submit for the review, you'll have to wait for the review process again. It's pivotal to note that if your changes aren't too big you could use [Ionic Deploy](http://blog.ionic.io/announcing-ionic-deploy-alpha-update-your-app-without-waiting/) to update your application without going through the review process, but more about this in the future tutorials.

# Conclusion
In this tutorial, we showed you how to prepare the application for the Apple's App Store and Google's Play Store, how to create the listings and finally how to publish the application to both stores. 

If you have any questions about this tutorial, please read the following help guides. Of course, I encourage you to ask a question in the comments so that anyone else who may have a similar question or doubt, learns something too.

With this, I leave you and hope you had a great learning experience with the series so far. We have few of the project ideas lined up already for the next posts. However, **I encourage you to share your wishes and ideas** about what kind of an app would you like to see built step by step. Remember, I'm here to help you get the best of Ionic framework so don't hesitate to ask.

# How to get help with Ionic framework
Few of the resources that proved indispensable when I was learning about Ionic:

+ For a quick framework reference, I'm suggesting the [official documentation](http://ionicframework.com/docs/), which is indeed very good.
+ If you're in search for a good book about Ionic, then you don't need to look further than [Ionic in Action: Hybrid Mobile Apps with Ionic and AngularJS](http://amzn.to/1EjwYNJ)
+ Of course, if you haven't, check out my first three posts in this series
+ One of the best resources on the net for programming related questions, about which you've no doubt heard, is [StackOverflow](http://stackoverflow.com/). You can view the specific Ionic tagged questions via [this link](http://stackoverflow.com/questions/tagged/ionic) or even [this one](http://stackoverflow.com/questions/tagged/ionic-framework). However, I urge you to please read their [help page](http://stackoverflow.com/help) before posting questions (especially if you're a new user to StackOverflow) because otherwise you may have a bad experience. If you're a new StackOverflow user, who's in a hurry to get acquainted with it, make sure you at least go through this [2-minute tour](http://stackoverflow.com/tour).
+ To get started with AngluarJS try [these](http://angular.codeschool.com/) [few](https://hackhands.com/finishing-Angular-TODO-application-deploying-production/) [resources](https://docs.angularjs.org/tutorial), not necessarily in that particular order.
+ Let me just kind of repeat myself by saying that if you have **any questions** about Ionic framework, or you had trouble following this tutorial (you couldn't install something), or you would like to suggest what kind of tutorials you would like to see regarding Ionic in the future, then please **share it in the comments** below. Also, you can reach me personally via @[HitmanHR](https://twitter.com/hitmanhr) or [my blog](http://www.nikola-breznjak.com/blog). I'll do my best to  answer all of your questions.
+ If, however, you favor "one on one" help, you can reach me [via HackHands](https://hackhands.com/dashboard/request/hitman666/preferred).