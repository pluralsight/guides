This tutorial will get you familiar with the fundamentals of setting up push notifications in your Android project using Firebase. 

## Firebase

Firebase serves as a module between your server and the devices that will be receiving the push notifications that you create. Your server informs Firebase that a notification has to be sent. Then Firebase does the work behind the scenes to get the notification published.

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/1-1024x662.png" alt="" width="1024" height="500" class="alignnone size-large wp-image-746" />

In order to establish connection with Firebase, you need to create a project for your own app in the <mark>Firebase console</mark>. You must set up your project in such a way that every time a user installs it, their device is registered in Firebase with a unique token. Although this may seem complex, the setup is actually simple.

## Create a Firebase project

Head to the [Firebase console][1], log in and create a new <b>Android</b> project. You will be asked to enter your package name. You can find your package name at the top of pretty much every Java file in your project. You can also find it in your `AndroidManifest.xml` file. (You can also add a SHA1 key for extra security measures, but that will not be required for this tutorial.)

Next, you will get a `google-services.json` file, which contains information about your project signature. In case you missed the instructions while creating your app, you need to add this file to your <b>app</b> directory, which you can find when you open your file structure to Project.

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/2.png" alt="" width="389" height="262" class="alignnone size-full wp-image-749" />

After that you need to add the <mark>gms</mark> dependency in your project-level <mark>build.gradle</mark> file:

```
buildscript {
dependencies {
// Add this line
 classpath 'com.google.gms:google-services:3.0.0'
    }
}
```
Then, add the corresponding plugin in your app-level <mark>build.gradle</mark>. You will also need to add a Firebase messaging dependency in this file.

```
dependencies {
compile 'com.google.firebase:firebase-messaging:10.2.1'
}
// Bottom of your file
apply plugin: 'com.google.gms.google-services'
```

<font color="red">Note that the `firebase-messaging` dependency <strong>must</strong> be the same version as other gms libraries. Otherwise there is a strong chance that your app will fail to register the device and obtain a Firebase token. There is an even greater chance that the application won't compile at all because it fails to find the Firebase classes that you're about to implement.</font>

## Creating the Firebase Services

The next step is to create two services. One will handle the device registration process, and the other will handle receiving the actual notifications.

Go to your `AndroidManifest.xml` file and add these service declarations under the <mark>application</mark> tag:

```
<service
		android:name=".notifications.MyFirebaseMessagingService"
		android:permission="com.google.android.c2dm.permission.SEND">
		<intent-filter>
			<action android:name="com.google.firebase.MESSAGING_EVENT" />
			<action android:name="com.google.android.c2dm.intent.RECEIVE" />
		</intent-filter>
	</service>
	<service android:name=".notifications.MyFirebaseInstanceIDService">
		<intent-filter>
			<action android:name="com.google.firebase.INSTANCE_ID_EVENT" />
		</intent-filter>
	</service>
```

Under the same tag you can also add metadata for default notification values, but it is not mandatory:

```
<meta-data
		android:name="com.google.firebase.messaging.default_notification_icon"
		android:resource="@mipmap/ic_launcher" />
	<meta-data
		android:name="com.google.firebase.messaging.default_notification_color"
		android:resource="@color/colorTransparent" />
		<meta-data android:name="com.google.android.gms.version"
		android:value="@integer/google_play_services_version" />
```

The last thing that you need to add to your manifest file is a RECEIVE permission:

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
```

## Setting up the services

Next, go ahead and create those two Java class services that you declared in the manifest in a new package called <mark>notifications</mark>.

This is the implementation of the `MyFirebaseInstanceIDService` class:

```
import android.content.SharedPreferences;
import android.preference.PreferenceManager;
import android.util.Log;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.FirebaseInstanceIdService;
import co.centroida.notifications.Constants;
public class MyFirebaseInstanceIDService extends FirebaseInstanceIdService {
private static final String TAG = "MyFirebaseIIDService";

@Override
public void onTokenRefresh() {
	// Get updated InstanceID token.
	String refreshedToken = FirebaseInstanceId.getInstance().getToken();
	Log.d(TAG, "Refreshed token: " + refreshedToken);
	// If you want to send messages to this application instance or
	// manage this apps subscriptions on the server side, send the
	// Instance ID token to your app server.
	SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
	preferences.edit().putString(Constants.FIREBASE_TOKEN, refreshedToken).apply();
    }
}
```

The purpose of this service is very simple:

It obtains a <mark>Firebase Token</mark>, thereby forging the connection between the device and Firebase. Through this token, you can send notifications to this specific device. When obtained, this token is saved in a shared preference for future use. Naturally, you would like to send it to your server at some point, say user registration, or even right away, so that the server can send this device notifications through Firebase.

Moving on to the more interesting class, namely `MyFirebaseMessagingService`.

```
public class MyFirebaseMessagingService extends FirebaseMessagingService {
@Override
public void onMessageReceived(RemoteMessage remoteMessage) {

   }
}
```
This service needs to extend FirebaseMessagingService. When the target device receives a notification, `onMessageReceived` is called. In your hands, you already have the `remoteMessage` object, which contains all the information that you received about the notification. Now let's create an actual notification with that information.
```
@Override public void onMessageReceived(RemoteMessage remoteMessage) {

	int notificationId = new Random().nextInt(60000);

	Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
	NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
			.setSmallIcon(R.drawable.ic_notification_small)  //a resource for your custom small icon
			.setContentTitle(remoteMessage.getData().get("title")) //the "title" value you sent in your notification
			.setContentText(remoteMessage.getData().get("message")) //ditto
			.setAutoCancel(true)  //dismisses the notification on click
			.setSound(defaultSoundUri);

	NotificationManager notificationManager =
			(NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

	notificationManager.notify(notificationId /* ID of notification */, notificationBuilder.build());

}
```
Here, to get unique notifications each time you receive a new message, for the sake of this example, we generate a random number and use it as notification ID. With this ID, you can do several things to your notifications. As such, you should probably group them if they are of the same kind, or update them. If you want to see each notification individually from the others, their IDs need to be different.

## Background compatibility

Also another thing that should be noted is that we use `remoteMessage.getData()` to access the values of the received notification. Now, there are two types of Firebase notifications - data messages and notification messages.

Data messages are handled here in `onMessageReceived` whether the app is in the foreground or background. However, notification messages are received only when the app is in foreground, making them kind of useless or at least rather boring, don't you think?

For an unified notifications system, we use messages that have a <b>data-only</b> payload.

<font color="red">Messages containing both notification and data payloads are treated as notification messages, so they won't be handled by `MyFirebaseMessagingService` when the app is in the background!</font>

If you haven't set up your server yet, you can still test your push notifications with a <mark>POST</mark> http request directly to Firebase. You can use any app you find fit for that, we use the Google <b>Postman</b> plugin. You can download it from [this link][2].

The endpoint you can use from firebase is this one:
<pre>https://fcm.googleapis.com/fcm/send
</pre>

## Obtaining the Server key: 

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/server_key-1024x323.png" alt="" width="1024" height="250" class="alignnone size-large wp-image-762" />

This request requires an Authorization header, the value of which is the <b>Server key</b> of your application in Firebase, preceded by a <mark>key=</mark>. You can find it in your Project settings in the [Firebase console][1] under the tab <b>Cloud messaging</b>.

This is what your <mark>POST</mark> request should look like: 

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/postman_header.png" alt="" width="967" height="221" class="alignnone size-full wp-image-754" />

And this should be the body of your request: 

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/postman_body-1.png" alt="" width="967" height="283" class="alignnone size-full wp-image-754" />

In the body of your request you need to specify the "<b>to</b>" field to send a notification to the targeted device. It's value is the aforementioned <b>Firebase token</b> that your device obtains in `MyFirebaseInstanceIDService`. You can retrieve it from the log message or directly from the shared preferences. In the data payload, you can specify all kinds of key-value pairs that would suit your application needs.

## Customizing notifications

Now that we can send and test notifications, we can make them fancier. First let's add a click functionality for the notification:

```
    Intent notificationIntent;
    if(StartActivity.isAppRunning){
    		notificationIntent = new Intent(this, ChildActivity.class);
    	}else{
    		notificationIntent = new Intent(this, StartActivity.class);
    	}
	notificationIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);

	final PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent,
			PendingIntent.FLAG_ONE_SHOT); 

	...

	Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
	NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
			.setSmallIcon(R.drawable.ic_notification_small)
			.setContentTitle(remoteMessage.getData().get("title"))
			.setContentText(remoteMessage.getData().get("message"))
			.setAutoCancel(true)
			.setSound(defaultSoundUri)
		    .setContentIntent(pendingIntent); 

```

It is recommended that you handle users clicking the notification while the app is still running. In this case we use a static boolean `isAppRunning` to determine whether the root activity (<mark>StartActivity</mark>) is running.

```
public class StartActivity extends AppCompatActivity {
public static boolean isAppRunning;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
    	setContentView(R.layout.activity_start);
    }
    @Override
    protected void onDestroy() {
    	super.onDestroy();
    	isAppRunning = false;
    }
}
```
You should consider what you want your notification intent to do so that it handles both cases without breaking your app's navigation structure.

### Adding an image

Pictures in notifications can be a great attention-grabber. This is how we can add an image to our push notification:

```
Bitmap bitmap = getBitmapfromUrl(remoteMessage.getData().get("image-url")); //obtain the image
Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
	NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
		    .setLargeIcon(bitmap)  //set it in the notification
			.setSmallIcon(R.mipmap.ic_launcher)
			.setContentTitle(remoteMessage.getData().get("title"))
			.setContentText(remoteMessage.getData().get("message"))
			.setAutoCancel(true)
			.setSound(defaultSoundUri)
			.setContentIntent(pendingIntent);
....
} 
//Simple method for image downloading
public Bitmap getBitmapfromUrl(String imageUrl) {
try {
		URL url = new URL(imageUrl);
		HttpURLConnection connection = (HttpURLConnection) url.openConnection();
		connection.setDoInput(true);
		connection.connect();
		InputStream input = connection.getInputStream();
		return BitmapFactory.decodeStream(input);

	} catch (Exception e) {
		e.printStackTrace();
		return null;
	}
} 
```
In this case we send an image URL in the notification payload for the app to download. Usually, such processes get executed on a separate thread. However, in this case, this class is a service, so once the code in `onMessageReceived` executes, the service, which is a thread different from the main thread, gets destroyed and with it goes every thread that was created by the service. Hence, we can afford to make the image download synchronously. This shouldn't pose a threat to performance, as the service thread is not the main thread.

### Notification Styles

The <mark>NotificationCompat.Builder</mark> supports a few different types of styles for notifications, including a player and ones with custom layouts:

```
Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
			.setLargeIcon(bitmap)
			.setSmallIcon(R.drawable.ic_notification_small)
			.setContentTitle(remoteMessage.getData().get("title"))
		    .setStyle(new NotificationCompat.BigPictureStyle()
					.setSummaryText(remoteMessage.getData().get("message"))
					.bigPicture(bitmap))
			.setContentText(remoteMessage.getData().get("message"))
			.setAutoCancel(true)
			.setSound(defaultSoundUri)
			.setContentIntent(pendingIntent);
```

### Notification buttons

You can also add buttons to your notifications:

```
Intent likeIntent = new Intent(this,LikeService.class);
likeIntent.putExtra(NOTIFICATION_ID_EXTRA,notificationId);
	likeIntent.putExtra(IMAGE_URL_EXTRA,remoteMessage.getData().get("image-url"));
	PendingIntent likePendingIntent = PendingIntent.getService(this,notificationId+1,likeIntent,PendingIntent.FLAG_ONE_SHOT); 

	Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
	NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this)
			.setLargeIcon(bitmap)
			.setSmallIcon(R.mipmap.ic_launcher)
			.setContentTitle(remoteMessage.getData().get("title"))
			.setStyle(new NotificationCompat.BigPictureStyle()
					.setSummaryText(remoteMessage.getData().get("message"))
					.bigPicture(bitmap))
			.setContentText(remoteMessage.getData().get("message"))
			.setAutoCancel(true)
			.setSound(defaultSoundUri)
            .addAction(R.drawable.ic_favorite_true,
                       getString(R.string.notification_like_button),likePendingIntent) //Setting the action
			.setContentIntent(pendingIntent);
```
In this case we use an action that is unrelated to the app and regardless of whether the app is active or inactive a certain simple action is executed through another simple service that extends the Service class:

```
public class LikeService extends Service {

    private static final String NOTIFICATION_ID_EXTRA = "notificationId";
    private static final String IMAGE_URL_EXTRA = "imageUrl";
    
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
    	//Saving action implementation
    	return null;
    }
}
```

The result of your efforts:

<img src="https://docs.centroida.co/wp-content/uploads/2017/05/notification.png" alt="" width="350" height="450" class="center-block size-full wp-image-758" />

That's all you need to get started with push notifications in Android!

Now you can have some fun exploring and styling your notifications. I hope this tutorial helped you out and you found it enjoyable. Until next time!

 [1]: https://console.firebase.google.com/
 [2]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=bg