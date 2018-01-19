In this tutorial we’ll create a ride-booking app with React Native and Pusher. The app that we’ll create will be similar to popular ride-booking apps like Uber, Lyft or Grab. 
React Native will be used to create an Android app for both the driver and the passenger. Pusher will be used for real-time communication between the two.


## What You’ll Create

Just like any other ride-booking app out there, there will be a driver app and a passenger app. The passenger app will be used for booking a ride, and the driver app simply receives any request that comes from the passenger app. For consistency purposes we’ll just refer to the app as “grabClone”.


## App Flow

The clone that we’re going to create will pretty much have the same flow as any ride booking app out there: passenger books a ride → app looks for a driver → driver accepts the request → driver picks up passenger → driver drives to destination → passenger pays the driver.

Here I just want to show you how this process would look like inside the app. That way you’ll have a clear picture of what you’re going to create.

1. The app determines the user’s location and shows it in a map (note: GPS needs to be enabled at this point). 

    ![Default screen](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500612181544_Screenshot_2017-07-20-10-30-07.png)

2. From the passenger app, the user clicks on “Book a Ride”. 
3. A modal will open that will allow the passenger to pick the place where they want to go. 

    ![Current location](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500612225092_Screenshot_2017-07-20-10-30-48.png)

4. The app asks the passenger to confirm their destination.

    ![Confirm destination](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500612322001_Screenshot_2017-07-20-10-31-03.png)

5. Once confirmed, the app sends a request to the driver app to pick up the passenger. A loading animation is shown while the app is waiting for a driver to accept the request.

    ![waiting for driver](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500612898340_Screenshot_2017-07-20-10-31-15.png)

6. The driver app receives the request. From here, the driver can either accept or reject the request.

    ![driver receives request](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500612408897_Screen+Shot+2017-07-20+at+10.34.21+AM.png)

7. Once the driver accepts the request, the driver’s details are shown in the passenger app.

    ![Driver details are sent to passenger](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500613002366_Screenshot_2017-07-20-10-32-39+1.png)

8. The passenger app shows the current location of the driver on the map.

    ![Current location of driver](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500613036704_Screenshot_2017-07-20-10-34-03.png)

9. Once the driver is within 50 meters of the passenger’s location, they will see an alert saying that the driver is near.
10. Once the driver is within 20 meters of the passenger’s location, the driver app sends a message to the passenger app that the driver is almost there.

    ![Notification near drop-off point](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500613085385_Screenshot_2017-07-20-10-35-43.png)

11. After picking up the passenger, the driver drives to their destination.
12. Once the driver is within 20 meters of their destination, the driver app sends a message to the passenger app that they’re very near their destination. 

    ![Notification 20 meters near drop-off point](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500613201701_Screenshot_2017-07-20-10-39-18.png)

At this point, the ride ends and the passenger can book another ride. The driver is also free to accept any incoming ride request.


## Prerequisites

- [**Pusher Account**](https://pusher.com/) - [signup for a Pusher account](https://dashboard.pusher.com/accounts/sign_up) or [login with your existing one](https://dashboard.pusher.com/accounts/sign_in). Once you’ve created an account, create a new app → select “React” for front-end technology → select “Node.js” for back-end technology. 

    ![create Pusher app](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500610327665_Screen+Shot+2017-07-21+at+12.11.41+PM.png)

Next, click on the “App Settings” tab and check “Enable client events”. This allows us to have the driver and passenger app directly communicate with each other. 

   ![Enable client events](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500610209176_Screen+Shot+2017-07-20+at+4.31.54+PM.png)

Last, click on the “App keys” and copy the credentials. If you’re worried about pricing, [the Pusher sandbox plan is pretty generous](https://dashboard.pusher.com/plans) so you can use it for free when testing the app.

- [**Install Android Studio**](https://developer.android.com/studio/index.html) - you don’t really need Android Studio, but it comes with Android SDK which is the one we need. Google also no longer offers a separate download for it. 

- [**Install React Native**](https://facebook.github.io/react-native/docs/getting-started.html) - the method I recommend for this is building projects the native way. When you’re on the React Native website, click on the “Building Projects with Native Code” tab and follow the instructions in there. The [expo](https://expo.io/) client is great for quickly prototyping apps but it doesn’t really offer a quick way for us to test the Geolocation features that we need for this app.

- [**Genymotion**](https://www.genymotion.com/fun-zone/) - for testing the driver app. We’re using this instead of the default Android emulator because it comes with a GPS simulation tool that allows us to search for a specific location and have it used as the location of the emulated device. It uses Google maps as the interface and you can move the marker as well. This allows us to simulate a moving vehicle.
  Once Genymotion is installed, you need to login to your account in order to add a device. For me I’ve installed Google Nexus 5x for testing. 

![Genymotion](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500610583449_Screen+Shot+2017-07-20+at+3.39.49+PM.png)

  
- **Android Device** - this will be used for testing the passenger app. Be sure to check the Android version of your phone. If it’s something as low as 4.2 then you’ll need to install additional packages via the Android SDK Manager. This is because, by default, React Native targets API version 23 or higher. That means the Android version of your phone needs to be version 6.0 at the very least or the app won’t run. If you have installed Android Studio, you can access the SDK Manager by opening Android Studio → click “Configure” → select “SDK Manager“. Then under the “SDK Platforms”, check the Android versions that you want to support. 

![Android SDK Manager](
https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500610476380_Screen+Shot+2017-07-20+at+3.34.11+PM.png)


While you’re there, click on the “SDK Tools” and make sure that you also have the same tools installed as mine:

![Android SDK Tools](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500610476415_Screen+Shot+2017-07-20+at+3.34.53+PM.png)

- **An extra computer** - this is optional. I just included it here because React Native can only run the app on a single device or emulator at a time. Thus, you need to do some extra work in order to run the two apps as you’ll see later on.


## Creating the Auth Server

Now it’s time to get our hands dirty. First, let’s work on the auth server. This is required because we will be sending [client events](https://pusher.com/docs/client_api_guide/client_events) from the app, client events requires the Pusher channel to be private, and private channels have restricted access. This is where the auth server comes in. It serves as a way for Pusher to know if a user that’s trying to connect is indeed a registered user of the app.

Start by installing the dependencies:

```
npm install --save express body-parser pusher
```

Next, create a `server.js` file and add the following code:

```javascript
var express = require('express');
var bodyParser = require('body-parser');
var Pusher = require('pusher');

var app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

var pusher = new Pusher({ // connect to pusher
  appId: process.env.APP_ID, 
  key: process.env.APP_KEY, 
  secret:  process.env.APP_SECRET,
  cluster: process.env.APP_CLUSTER, 
});

app.get('/', function(req, res){ // for testing if the server is running
  res.send('all is well...');
});

// for authenticating users
app.get("/pusher/auth", function(req, res) {
  var query = req.query;
  var socketId = query.socket_id;
  var channel = query.channel_name;
  var callback = query.callback;

  var auth = JSON.stringify(pusher.authenticate(socketId, channel));
  var cb = callback.replace(/\"/g,"") + "(" + auth + ");";

  res.set({
    "Content-Type": "application/javascript"
  });

  res.send(cb);
});

app.post('/pusher/auth', function(req, res) {
  var socketId = req.body.socket_id;
  var channel = req.body.channel_name;
  var auth = pusher.authenticate(socketId, channel);
  res.send(auth);
});

var port = process.env.PORT || 5000;
app.listen(port);
```

I’m no longer going to go into detail what the code above does since its already explained in the docs for [Authenticating Users](https://pusher.com/docs/authenticating_users). 
To keep things simple, I haven’t actually added the code to check if a user really exists in a database. You can do that in the `/pusher/auth`  endpoint by checking if a username exists. Here’s an example:

```javascript
var users = ['luz', 'vi', 'minda'];
var username = req.body.username;

if(users.indexOf(username) !== -1){
  var socketId = req.body.socket_id;
  var channel = req.body.channel_name;
  var auth = pusher.authenticate(socketId, channel);
  res.send(auth);
}

// otherwise: return error
```

Don’t forget to pass in the `username` when connecting to Pusher on the client-side later on.

Try running the server once that’s done:

```
node server.js
```

Access `http://localhost:5000` on your browser to see if it works.


## Deploying the Auth Server

Since Pusher will have to connect to the auth server, it needs to be accessible from the internet. 
You can use [now.sh](https://zeit.co/now) to deploy the auth server. You can install it with the following command:

```
npm install now
```

Once installed, you can now navigate to the folder where you have the `server.js` file and execute `now`. You’ll be asked to enter your email and verify your account. 

Once your account is verified, execute the following to add your Pusher app settings as environment variables to your now.sh account so you can use it from inside the server:

```
now secret add pusher_app_id YOUR_PUSHER_APP_ID
now secret add pusher_app_key YOUR_PUSHER_APP_KEY
now secret add pusher_app_secret YOUR_PUSHER_APP_SECRET
now secret add pusher_app_cluster YOUR_PUSHER_APP_CLUSTER
```

Next, deploy the server while supplying the secret values that you’ve added:

```
now -e APP_ID=@pusher_app_id -e APP_KEY=@pusher_app_key -e APP_SECRET=@pusher_app_secret APP_CLUSTER=@pusher_app_cluster
```

This allows you to access your Pusher app settings from inside the server like so:

```
process.env.APP_ID
```

The deploy URL that now.sh returns is the URL that you’ll use later on to connect the app to the auth server.

## Creating the Driver App

Now you’re ready to start creating the driver app. 

First, create a new React Native app:

```
react-native init grabDriver
```

**Installing the Dependencies**

Once that’s done, navigate inside the `grabDriver`  directory and install the libraries that we’ll need. This includes [pusher-js](https://github.com/pusher/pusher-js) for working with Pusher, [React Native Maps](https://github.com/airbnb/react-native-maps/) for displaying a map, and [React Native Geocoding](https://github.com/marlove/react-native-geocoding) for reverse-geocoding coordinates to the actual name of a place: 

```
npm install --save pusher-js react-native-maps react-native-geocoding
```

Once all the libraries are installed, React Native Maps needs some additional steps in order for it to work. First is linking the project resources:

```
react-native link react-native-maps
```

Next, you need to create a Google project, get an API key from the [Google developer console](https://console.developers.google.com/apis/credentials), and enable the [Google Maps Android API](https://console.developers.google.com/apis/api/maps-android-backend.googleapis.com/overview) and [Google Maps Geocoding API](https://console.developers.google.com/apis/api/geocoding-backend.googleapis.com/overview). After that, open the `android\app\src\main\AndroidManifest.xml` file in your project directory. Under the `<application>` tag, add a `<meta-data>` containing the server API key.

```xml
<application>
    <meta-data
      android:name="com.google.android.geo.API_KEY"
      android:value="YOUR GOOGLE SERVER API KEY"/>
</application>
```

While you’re there, add the following below the default permissions. This allows us to check for network status and request for Geolocation data from the device.

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Also make sure that its targeting the same API version as the device you installed with Genymotion. As I’ve said earlier, if its version 23 or above you won’t really need to do anything, but if its lower than that then it has to be exact for the app to work.

```xml
<uses-sdk
        android:minSdkVersion="16"
        android:targetSdkVersion="23" />
```

Lastly, since we’ll be primarily using Genymotion for testing the driver app, you need to follow the [instructions here](https://www.genymotion.com/help/desktop/faq/#google-play-services). In case the link goes broken, here’s what you need to do:


1. Visit [opengapps.org](http://opengapps.org/).
2. Select x86 as platform.
3. Choose the Android version corresponding to your virtual device.
4. Select nano as variant.
5. Download the zip file.
6. Drag & Drop the zip installer in new Genymotion virtual device (2.7.2 and above only).
7. Follow the pop-up instructions.

We need to do this because the React Native Maps library primarily uses Google Maps. We need to add Google Play Services in order for it to work. Unlike most Android phones which already comes with this installed, Genymotion doesn’t have it by default due to intellectual property reasons. Thus, we need to manually install it.

If you’re reading this some time after it was published, be sure to check out the [Installation docs](https://github.com/airbnb/react-native-maps/blob/master/docs/installation.md) to make sure you’re not missing anything.

**Coding the Driver App**

Now you’re ready to start coding the app. Start by opening the `index.android.js` file and replace the default code with the following:

```javascript
import { AppRegistry } from 'react-native';
import App from './App';
AppRegistry.registerComponent('grabDriver', () => App);
```

What this does is importing the `App` component which is the main component for the app. It is then registered as the default component so it will be rendered in the screen.

Next, create the `App.js` file and import the things we need from the React Native package:

```javascript
import React, { Component } from 'react';
import {
  StyleSheet,
  Text,
  View,
  Alert
} from 'react-native';
```

Also import the third-party libraries that we installed earlier:

```javascript
import Pusher from 'pusher-js/react-native';
import MapView from 'react-native-maps';

import Geocoder from 'react-native-geocoding';
Geocoder.setApiKey('YOUR GOOGLE SERVER API KEY');
```

Lastly, import the `helpers` file:

```javascript
import { regionFrom, getLatLonDiffInMeters } from './helpers';
```

The `helpers.js` file contains the following:

```javascript
export function regionFrom(lat, lon, accuracy) {
  const oneDegreeOfLongitudeInMeters = 111.32 * 1000;
  const circumference = (40075 / 360) * 1000;

  const latDelta = accuracy * (1 / (Math.cos(lat) * circumference));
  const lonDelta = (accuracy / oneDegreeOfLongitudeInMeters);

  return {
    latitude: lat,
    longitude: lon,
    latitudeDelta: Math.max(0, latDelta),
    longitudeDelta: Math.max(0, lonDelta)
  };
} 

export function getLatLonDiffInMeters(lat1, lon1, lat2, lon2) {
  var R = 6371; // Radius of the earth in km
  var dLat = deg2rad(lat2-lat1);  // deg2rad below
  var dLon = deg2rad(lon2-lon1); 
  var a = 
    Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.cos(deg2rad(lat1)) * Math.cos(deg2rad(lat2)) * 
    Math.sin(dLon/2) * Math.sin(dLon/2)
    ; 
  var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a)); 
  var d = R * c; // Distance in km
  return d * 1000;
}

function deg2rad(deg) {
  return deg * (Math.PI/180)
}
```

These functions are used for getting the latitude and longitude delta values needed by the React Native Maps library to display a map. The other function (`getLatLonDiffInMeters`) is used for determining the distance in meters between two coordinates. Later on, this will allow us to inform the user’s whether they’re already near each other or when they’re near their destination.

Next, create the main app component and declare the default states:

```javascript
export default class grabDriver extends Component {

  state = {
    passenger: null, // for storing the passenger info
    region: null, // for storing the current location of the driver
    accuracy: null, // for storing the accuracy of the location
    nearby_alert: false, // whether the nearby alert has already been issued
    has_passenger: false, // whether the driver has a passenger (once they agree to a request, this becomes true)
    has_ridden: false // whether the passenger has already ridden the vehicle
  }
}
// next: add constructor code
```

Inside the constructor,  initialize the variables that will be used throughout the app:

```javascript
constructor() {
  super();

  this.available_drivers_channel = null; // this is where passengers will send a request to any available driver
  this.ride_channel = null; // the channel used for communicating the current location
  // for a specific ride. Channel name is the username of the passenger
 
  this.pusher = null; // the pusher client
}

// next: add code for connecting to pusher
```

Before the component is mounted, connect to the auth server that you created earlier. Be sure to replace the values for the pusher key, `authEndpoint` and `cluster`.

```javascript
componentWillMount() {
  this.pusher = new Pusher('YOUR PUSHER KEY', {
    authEndpoint: 'YOUR PUSHER AUTH SERVER ENDPOINT',
    cluster: 'YOUR PUSHER CLUSTER',
    encrypted: true
  });
  
  // next: add code for listening to passenger requests
}
```

Now that you’ve connected to the auth server, you can now start listening for requests coming from the passenger app. The first step is to subscribe to a [private channel](https://pusher.com/docs/client_api_guide/client_private_channels). This channel is where all passengers and drivers subscribe to. In this case, its used by drivers to listen for ride requests. It needs to be a private channel because [client events](https://pusher.com/docs/client_api_guide/client_events#trigger-events) can only be triggered on private and presence channels due to security reasons. You know that it’s a private channel because of the `private-`  prefix. 

```javascript
this.available_drivers_channel = this.pusher.subscribe('private-available-drivers'); // subscribe to "available-drivers" channel
```

Next, listen to the `client-driver-request` event. You know that this is a client event because of the `client-` prefix. Client events doesn’t need server intervention in order to work, the messages are sent directly to from client to client. That’s the reason why we need an auth server to make sure all the users that are trying to connect are real users of the app.

Going back to the code, we listen for client events by calling the `bind` method on the channel that we subscribed to and passing in the name of the event as the first argument. The second argument is the function that you want to execute once this event is triggered from another client (from anyone using the passenger app to request a ride). In the code below, we show an alert message asking the driver if they want to accept the passenger. Note that the app assumes that there can only be one passenger at any single time.

```javascript
// listen to the "driver-request" event
this.available_drivers_channel.bind('client-driver-request', (passenger_data) => {
  
  if(!this.state.has_passenger){ // if the driver has currently no passenger
    // alert the driver that they have a request
    Alert.alert(
      "You got a passenger!", // alert title
      "Pickup: " + passenger_data.pickup.name + "\nDrop off: " + passenger_data.dropoff.name, // alert body
      [
        {
          text: "Later bro", // text for rejecting the request
          onPress: () => {
            console.log('Cancel Pressed');
          },
          style: 'cancel'
        },
        {
          text: 'Gotcha!', // text for accepting the request
          onPress: () => {
            // next: add code for when driver accepts the request
          }  
        },
      ],
      { cancelable: false } // no cancel button
    );

  }

});
```

Once the driver agrees to pick up the passenger, we subscribe to their private channel. This channel is reserved only for communication between the driver and the passenger, that’s why we’re using the unique passenger username as part of the channel’s name.

```javascript
this.ride_channel = this.pusher.subscribe('private-ride-' + passenger_data.username);
```

Not unlike the `available-drivers` channel, we’ll need to listen for when the subscription actually succeeded (`pusher:subscription_succeeded`) before we do anything else. This is because we’re going to immediately trigger a client event to be sent to the passenger. This event (`client-driver-response`) is a handshake event to let the passenger know that the driver they sent their request to is still available. If the passenger still hasn’t gotten a ride at that time, the passenger app triggers the same event to let the driver know that they’re still available for picking up. At this point, we update the state so that the UI changes accordingly.

```javascript
this.ride_channel.bind('pusher:subscription_succeeded', () => {
   // send a handshake event to the passenger
  this.ride_channel.trigger('client-driver-response', {
    response: 'yes' // yes, I'm available
  });
  
  // listen for the acknowledgement from the passenger
  this.ride_channel.bind('client-driver-response', (driver_response) => {
    
    if(driver_response.response == 'yes'){ // passenger says yes

      //passenger has no ride yet
      this.setState({
        has_passenger: true,
        passenger: {
          username: passenger_data.username,
          pickup: passenger_data.pickup,
          dropoff: passenger_data.dropoff
        }
      });
      
      // next: reverse-geocode the driver location to the actual name of the place
      
    }else{
      // alert that passenger already has a ride
      Alert.alert(
        "Too late bro!",
        "Another driver beat you to it.",
        [
          {
            text: 'Ok'
          },
        ],
        { cancelable: false }
      );
    }

  });

});
```

Next, we use the Geocoding library to determine the name of the place where the driver is currently at. Behind the scenes, this uses the Google Geocoding API and it usually returns the street name. Once we get a response back, we trigger the `found-driver` event to let the passenger know that the app has found a driver for them. This contains driver info such as the name and the current location.

```javascript
Geocoder.getFromLatLng(this.state.region.latitude, this.state.region.longitude).then(
  (json) => {
    var address_component = json.results[0].address_components[0];
    
    // inform passenger that it has found a driver
    this.ride_channel.trigger('client-found-driver', { 
      driver: {
        name: 'John Smith'
      },
      location: { 
        name: address_component.long_name,
        latitude: this.state.region.latitude,
        longitude: this.state.region.longitude,
        accuracy: this.state.accuracy
      }
    });

  },
  (error) => {
    console.log('err geocoding: ', error);
  }
);  
// next: add componentDidMount code
```

Once the component is mounted, we use [React Native’s Geolocation API](https://facebook.github.io/react-native/docs/geolocation.html) to watch for location updates. The function that you pass to the `watchPosition` function gets executed everytime the location changes.

```javascript
componentDidMount() {
  this.watchId = navigator.geolocation.watchPosition(
    (position) => {
     
      var region = regionFrom(
        position.coords.latitude, 
        position.coords.longitude, 
        position.coords.accuracy
      );
      // update the UI
      this.setState({
        region: region,
        accuracy: position.coords.accuracy
      });
      
      if(this.state.has_passenger && this.state.passenger){
        // next: add code for sending driver's current location to passenger
      }
    },
    (error) => this.setState({ error: error.message }),
    { 
      enableHighAccuracy: true, // allows you to get the most accurate location
      timeout: 20000, // (milliseconds) in which the app has to wait for location before it throws an error
      maximumAge: 1000, // (milliseconds) if a previous location exists in the cache, how old for it to be considered acceptable 
      distanceFilter: 10 // (meters) how many meters the user has to move before a location update is triggered
    },
  );
}
```

Next, send the driver’s current location to the passenger. This will update the UI on the passenger app to show the current location of the driver. You’ll see how the passenger app binds to this event later on when we move on to coding the passenger app.

```javascript
this.ride_channel.trigger('client-driver-location', { 
  latitude: position.coords.latitude,
  longitude: position.coords.longitude,
  accuracy: position.coords.accuracy
});
```

Next, we want to inform both the passenger and the driver that they’re already near each other. For that we use the `getLatLonDiffInMeters` function from the `helpers.js` file in order to determine the number of meters between the passenger and the driver. Since the driver already received the passenger location when they accepted the request, it’s only a matter of getting the current location of the driver and passing it to the `getLanLonDiffInMeters` function to get the difference in meters. From there, we simply inform the driver or the passenger based on the number of meters. Later on you’ll see how these events are received in the passenger app.

```javascript
var diff_in_meter_pickup = getLatLonDiffInMeters(
  position.coords.latitude, position.coords.longitude, 
  this.state.passenger.pickup.latitude, this.state.passenger.pickup.longitude);

if(diff_in_meter_pickup <= 20){
  
  if(!this.state.has_ridden){
    // inform the passenger that the driver is very near
    this.ride_channel.trigger('client-driver-message', {
      type: 'near_pickup',
      title: 'Just a heads up',
      msg: 'Your driver is near, let your presence be known!'
    });

    /*
    we're going to go ahead and assume that the passenger has rode 
    the vehicle at this point
    */
    this.setState({
      has_ridden: true
    });
  }

}else if(diff_in_meter_pickup <= 50){
  
  if(!this.state.nearby_alert){
    this.setState({
      nearby_alert: true
    });
    /* 
    since the location updates every 10 meters, this alert will be triggered 
    at least five times unless we do this
    */
    Alert.alert(
      "Slow down",
      "Your passenger is just around the corner",
      [
        {
          text: 'Gotcha!'
        },
      ],
      { cancelable: false }
    );

  }

}

// next: add code for sending messages when near the destination
```

At this point, we assume that the driver has picked up the passenger and that they’re now heading to their destination. So this time we get the distance between the current location and the drop-off point. Once they’re 20 meters to the drop-off point, the driver app sends a message to the passenger that they’re very close to their destination. Once that’s done, we assume that the passenger will get off in a few seconds. So we unbind the events that we’re listening to and unsubscribe from the passenger’s private channel. This effectively cuts the connection between the driver and passenger app. The only connection that stays open is the `available-drivers` channel.

```javascript
var diff_in_meter_dropoff = getLatLonDiffInMeters(
  position.coords.latitude, position.coords.longitude, 
  this.state.passenger.dropoff.latitude, this.state.passenger.dropoff.longitude);

if(diff_in_meter_dropoff <= 20){
  this.ride_channel.trigger('client-driver-message', {
    type: 'near_dropoff',
    title: "Brace yourself",
    msg: "You're very close to your destination. Please prepare your payment."
  });

  // unbind from passenger event
  this.ride_channel.unbind('client-driver-response');
  // unsubscribe from passenger channel 
  this.pusher.unsubscribe('private-ride-' + this.state.passenger.username);

  this.setState({
    passenger: null,
    has_passenger: false,
    has_ridden: false
  });

}

// next: add code for rendering the UI
```

The UI for the driver app only displays the map and the markers for the driver and passenger.

```javascript
render() {
  return (
    <View style={styles.container}>
      {
        this.state.region && 
        <MapView
          style={styles.map}
          region={this.state.region}
        >
            <MapView.Marker
              coordinate={{
              latitude: this.state.region.latitude, 
              longitude: this.state.region.longitude}}
              title={"You're here"}
            />
            {
              this.state.passenger && !this.state.has_ridden && 
              <MapView.Marker
                coordinate={{
                latitude: this.state.passenger.pickup.latitude, 
                longitude: this.state.passenger.pickup.longitude}}
                title={"Your passenger is here"}
                pinColor={"#4CDB00"}
              />
            }
        </MapView>
      }
    </View>
  );
}
// next: add code when component unmounts
```

Before the component unmounts, we stop the location watcher by calling the `clearWatch` method:

```javascript
componentWillUnmount() {
  navigator.geolocation.clearWatch(this.watchId);
} 
```

Lastly, add the styles:

```javascript
const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'flex-end',
    alignItems: 'center',
  },
  map: {
    ...StyleSheet.absoluteFillObject,
  },
});
```

## Creating the Passenger App

The passenger app is going to be pretty similar to the driver app so I’ll no longer go into detail on parts that are similar. Go ahead and create a  new app:

```
react-native init grabClone
```

**Installing the Dependencies**

You’d also need to install the same libraries plus a couple more:

```
npm install --save pusher-js react-native-geocoding github:geordasche/react-native-google-place-picker react-native-loading-spinner-overlay react-native-maps
```

The other two libraries are [Google Place Picker](https://github.com/q6112345/react-native-google-place-picker) and [Loading Spinner Overlay](https://github.com/joinspontaneous/react-native-loading-spinner-overlay). Though we’ve used a [fork](https://github.com/geordasche/react-native-google-place-picker) of the Google Place Picker because of a compatibility issue with React Native Maps that wasn’t fixed in the original repo yet.

Since we’ve installed the same libraries, you can go back to the section where we did some additional configuration in order for the library to work. Come back here once you’ve done those.

Next, the Google Place Picker also needs some additional configuration for it to work. First, open the `android/app/src/main/java/com/grabClone/MainApplication.java` file and add the following below the last import:

```java
import com.reactlibrary.RNGooglePlacePickerPackage;
```
 
 Add the library that you just imported under the `getPackages()` function. While you’re there, also make sure that the `MapsPackage()` is listed as well.
 
```java
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
      new MainReactPackage(),
      new MapsPackage(),
      new RNGooglePlacePickerPackage() // <- add this
  );
}
```

Next, open the `android/settings.gradle` file and add these right above the `include ':app'` directive:

```
include ':react-native-google-place-picker'
project(':react-native-google-place-picker').projectDir = new File(rootProject.projectDir,         '../node_modules/react-native-google-place-picker/android')
```

While you’re there, also make sure that the resources for React Native Maps are also added:

```
include ':react-native-maps'
project(':react-native-maps').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-maps/lib/android')
```

Next, open the `android/app/build.gradle` file and add the following under the `dependencies`:

```
dependencies {
  compile project(':react-native-google-place-picker') // <- add this
}
```

Lastly, make sure that React Native Maps is also compiled:

```
compile project(':react-native-maps')
```

**Coding the Passenger App**

Open the `index.android.js` file and add the following:

```javascript
import { AppRegistry } from 'react-native';
import App from './App';
AppRegistry.registerComponent('grabClone', () => App);
```

Just like the driver app, it also uses `App.js` as the main component. Go ahead and import the libraries. It also uses the same `helpers.js` file so you can copy it from the driver app as well.

```javascript
import React, { Component } from 'react';
import { StyleSheet, Text, View, Button, Alert } from 'react-native';

import Pusher from 'pusher-js/react-native';
import RNGooglePlacePicker from 'react-native-google-place-picker';
import Geocoder from 'react-native-geocoding';
import MapView from 'react-native-maps';
import Spinner from 'react-native-loading-spinner-overlay';

import { regionFrom, getLatLonDiffInMeters } from './helpers'; 

Geocoder.setApiKey('YOUR GOOGLE SERVER API KEY');
```

Create the component and declare the default states:

```javascript
export default class App extends Component {
  state = {
    location: null, // current location of the passenger
    error: null, // for storing errors
    has_ride: false, // whether the passenger already has a driver which accepted their request
    destination: null, // for storing the destination / dropoff info
    driver: null, // the driver info
    origin: null, // for storing the location where the passenger booked a ride
    is_searching: false, // if the app is currently searching for a driver
    has_ridden: false // if the passenger has already been picked up by the driver
  };
  
  // next: add constructor code
}
```

To keep things simple, we declare the username of the passenger in the constructor. We also initialize the Pusher channels:

```javascript
constructor() {
  super();
  this.username = 'wernancheta'; // the unique username of the passenger
  this.available_drivers_channel = null; // the pusher channel where all drivers and passengers are subscribed to
  this.user_ride_channel = null; // the pusher channel exclusive to the passenger and driver in a given ride
  this.bookRide = this.bookRide.bind(this); // bind the function for booking a ride
}
// next: add bookRide() function
```

The `bookRide()` function gets executed when the user taps on the “Book Ride” button. This opens a place picker which allows the user to pick their destination. Once a location is picked, the app sends a ride request to all drivers. As you have seen in the driver app earlier, this triggers an alert to show in the driver app which asks if the driver wants to accept the request or not. At this point the loader will keep on spinning until a driver accepts the request.

```javascript
bookRide() {

  RNGooglePlacePicker.show((response) => {
    if(response.didCancel){
      console.log('User cancelled GooglePlacePicker');
    }else if(response.error){
      console.log('GooglePlacePicker Error: ', response.error);
    }else{
      this.setState({
        is_searching: true, // show the loader
        destination: response // update the destination, this is used in the UI to display the name of the place
      });
      
      // the pickup location / origin
      let pickup_data = {
        name: this.state.origin.name,
        latitude: this.state.location.latitude,
        longitude: this.state.location.longitude
      };
      
      // the dropoff / destination
      let dropoff_data = {
        name: response.name,
        latitude: response.latitude,
        longitude: response.longitude
      };
      
      // send a ride request to all drivers
      this.available_drivers_channel.trigger('client-driver-request', {
        username: this.username,
        pickup: pickup_data,
        dropoff: dropoff_data
      });

    }
  });
}
// next: add _setCurrentLocation() function
```

The `_setCurrentLocation()` function gets the passenger’s current location. Note that here we’re using  `getCurrentPosition()` as opposed to `watchPosition()` which we used in the driver app earlier. The only difference between the two is that `getCurrentPosition()` only gets the location once. 

```javascript
_setCurrentLocation() {

  navigator.geolocation.getCurrentPosition(
    (position) => {
      var region = regionFrom(
        position.coords.latitude, 
        position.coords.longitude, 
        position.coords.accuracy
      );
      
      // get the name of the place by supplying the coordinates      
      Geocoder.getFromLatLng(position.coords.latitude, position.coords.longitude).then(
        (json) => {
          var address_component = json.results[0].address_components[0];
          
          this.setState({
            origin: { // the passenger's current location
              name: address_component.long_name, // the name of the place
              latitude: position.coords.latitude,
              longitude: position.coords.longitude
            },
            location: region, // location to be used for the Map
            destination: null, 
            has_ride: false, 
            has_ridden: false,
            driver: null    
          });

        },
        (error) => {
          console.log('err geocoding: ', error);
        }
      );

    },
    (error) => this.setState({ error: error.message }),
    { enableHighAccuracy: false, timeout: 10000, maximumAge: 3000 },
  );

}

// next: add componentDidMount() function
```

When the component mounts, we want to set the current location of the passenger, connect to the auth server and subscribe to the two channels: available drivers and the passenger’s private channel for communicating only with the driver’s where the ride request was sent to.

```javascript
componentDidMount() {

  this._setCurrentLocation(); // set current location of the passenger
  // connect to the auth server
  var pusher = new Pusher('YOUR PUSHER API KEY', {
    authEndpoint: 'YOUR AUTH SERVER ENDPOINT',
    cluster: 'YOUR PUSHER CLUSTER',
    encrypted: true
  });
  
  // subscribe to the available drivers channel
  this.available_drivers_channel = pusher.subscribe('private-available-drivers');
  
  // subscribe to the passenger's private channel
  this.user_ride_channel = pusher.subscribe('private-ride-' + this.username);
  
  // next: add code for listening to handshake responses
  
}
```

Next, add the code for listening to the handshake response by the driver. This is being sent from the driver app when the driver accepts a ride request. This allows us to make sure that the passenger is still looking for a ride. If the passenger responds with “yes” then that’s the only time that the driver sends their information.

```javascript
this.user_ride_channel.bind('client-driver-response', (data) => {
  let passenger_response = 'no';
  if(!this.state.has_ride){ // passenger is still looking for a ride
    passenger_response = 'yes';
  }

  // passenger responds to driver's response
  this.user_ride_channel.trigger('client-driver-response', {
    response: passenger_response
  });
});

// next: add listener for when a driver is found
```

The driver sends their information by triggering the `client-found-driver` event. As you have seen in the driver app earlier, this contains the name of the driver as well as their current location.

```javascript
this.user_ride_channel.bind('client-found-driver', (data) => {
  // the driver's location info  
  let region = regionFrom(
    data.location.latitude,
    data.location.longitude,
    data.location.accuracy 
  );

  this.setState({
    has_ride: true, // passenger has already a ride
    is_searching: false, // stop the loading UI from spinning
    location: region, // display the driver's location in the map
    driver: { // the driver location details
      latitude: data.location.latitude,
      longitude: data.location.longitude,
      accuracy: data.location.accuracy
    }
  });
  
  // alert the passenger that a driver was found
  Alert.alert(
    "Orayt!",
    "We found you a driver. \nName: " + data.driver.name + "\nCurrent location: " + data.location.name,
    [
      {
        text: 'Sweet!'
      },
    ],
    { cancelable: false }
  );      
});
// next: add code for listening to driver's current location
```

At this point, the passenger can now listen to location changes from the driver. We simply update the UI everytime this event is triggered:

```javascript
this.user_ride_channel.bind('client-driver-location', (data) => {
  let region = regionFrom(
    data.latitude,
    data.longitude,
    data.accuracy
  );
  
  // update the Map to display the current location of the driver
  this.setState({
    location: region, // the driver's location
    driver: {
      latitude: data.latitude,
      longitude: data.longitude
    }
  });

});
```

Next is the event that is triggered on specific instances. It’s main purpose is to send updates to the passenger regarding the location of the driver (`near_pickup` ) and also when they’re already near the drop-off location (`near_dropoff`). 

```javascript
this.user_ride_channel.bind('client-driver-message', (data) => {
  if(data.type == 'near_pickup'){ // the driver is very near the pickup location
    // remove passenger marker since we assume that the passenger has rode the vehicle at this point
    this.setState({
      has_ridden: true 
    });
  }

  if(data.type == 'near_dropoff'){ // they're near the dropoff location
    this._setCurrentLocation(); // assume that the ride is over, so reset the UI to the current location of the passenger
  }
  
  // display the message sent from the driver app
  Alert.alert(
    data.title,
    data.msg,
    [
      {
        text: 'Aye sir!'
      },
    ],
    { cancelable: false }
  );        
});

// next: render the UI
```

The UI composed of the loading spinner (only visible when the app is searching for a driver), the header, the button for booking a ride, the passenger location (`origin`) and their destination, and the map which initially displays the current location of the user and then displays the current location of the driver once a ride has been booked.

```javascript
render() {

  return (
    <View style={styles.container}>
      <Spinner 
          visible={this.state.is_searching} 
          textContent={"Looking for drivers..."} 
          textStyle={{color: '#FFF'}} />
      <View style={styles.header}>
        <Text style={styles.header_text}>GrabClone</Text>
      </View>
      {
        !this.state.has_ride && 
        <View style={styles.form_container}>
          <Button
            onPress={this.bookRide}
            title="Book a Ride"
            color="#103D50"
          />
        </View>
      }
      
      <View style={styles.map_container}>  
      {
        this.state.origin && this.state.destination &&
        <View style={styles.origin_destination}>
          <Text style={styles.label}>Origin: </Text>
          <Text style={styles.text}>{this.state.origin.name}</Text>
         
          <Text style={styles.label}>Destination: </Text>
          <Text style={styles.text}>{this.state.destination.name}</Text>
        </View>  
      }
      {
        this.state.location &&
        <MapView
          style={styles.map}
          region={this.state.location}
        >
          {
            this.state.origin && !this.state.has_ridden &&
            <MapView.Marker
              coordinate={{
              latitude: this.state.origin.latitude, 
              longitude: this.state.origin.longitude}}
              title={"You're here"}
            />
          }
  
          {
            this.state.driver &&
            <MapView.Marker
              coordinate={{
              latitude: this.state.driver.latitude, 
              longitude: this.state.driver.longitude}}
              title={"Your driver is here"}
              pinColor={"#4CDB00"}
            />
          }
        </MapView>
      }
      </View>
    </View>
  );
}
```

Lastly, add the styles:

```javascript
const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'flex-end'
  },
  form_container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20
  },
  header: {
    padding: 20,
    backgroundColor: '#333',
  },
  header_text: {
    color: '#FFF',
    fontSize: 20,
    fontWeight: 'bold'
  },  
  origin_destination: {
    alignItems: 'center',
    padding: 10
  },
  label: {
    fontSize: 18
  },
  text: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  map_container: {
    flex: 9
  },
  map: {
   flex: 1
  },
});
```

## Running the App

Now you’re ready to run the app. As I mentioned in the **Prerequisites** section earlier, you’re going to optionally need two machines, one for running each of the app. This will allow you to enable logging (`console.log`) for both. But if you have only one machine then you have to run them in particular order: passenger app first and then driver app.

Go ahead and connect your Android device to your computer and run the following command:

```
react-native run-android
```

This will compile, install and run the app on your device. Once its running, terminate the watcher and disconnect your device from the computer. 

![compile app](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500609946898_Screen+Shot+2017-07-21+at+12.04.31+PM.png)

Next, open Genymotion and launch the device that you installed earlier. This time, run the driver app. Once the app runs you’ll see a blank screen. This is normal because the app needs a location in order to render something. You can do that by clicking on “GPS” located on the upper right-side of the emulator UI then enable GPS. 

![enable GPS on Genymotion](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500608355479_Screen+Shot+2017-07-21+at+7.35.46+AM.png)

You can also click on the map button and select a specific location if you want:

![Change location](https://d2mxuefqeaa7sj.cloudfront.net/s_6E04D0395F3E9633E1AA8A9CFF9F8EA297AF4AA1392D072366B2CFB9DE43A3C7_1500608476706_Screen+Shot+2017-07-21+at+7.39.41+AM.png)

Once you’ve selected a location, the map UI in the app should show the same location that you selected.

Next, you can now follow the steps on the **App Flow** section earlier.  Note that you can emulate a moving vehicle by clicking around the Genymotion Map UI. If a passenger has already booked a ride and the driver has accepted the request, it should start updating both the passenger app and the driver app of the current location of the driver.

If you’re using two machines, then you can simply run `react-native run-android` on both. One should be connected to your device and the other should have the Genymotion emulator open.


## Conclusion

That’s it! In this tutorial you’ve learned how to make use of Pusher to create a ride-booking app. As you have seen, the app that you’ve built is pretty bare-bones. We’ve only stick to building the most important parts of a ride-booking app. If you want you can add more features to the app and maybe use it on your own projects. You can find the source code used in this app on its [Github repo](https://github.com/anchetaWern/grabClone).

*Originally published on the [Pusher blog](https://blog.pusher.com/creating-ride-booking-app-react-native-pusher/).*