## Creating the project

```
$ npm install -g ionic@beta 
$ npm install -g cordova
```

Create the project image-uploader

```
$ ionic start image-uploader blank --v2 --ts
```

## Installing Plugins

We want to have some images to upload so we are going to integrate the Camera here to get a photo or to pick one from the gallery. In ionic2, we get Ionic-Native by default so we should be good to go here.

- [Documentation for cordova-plugin-camera](https://github.com/apache/cordova-plugin-camera)
- [Documentation for using Camera with Ionic-Native](http://ionicframework.com/docs/v2/native/camera/)

```
$ ionic plugin add cordova-plugin-camera
```

## Save Project State
Lets save the project state so when you check it out later, everything is good to go
```
$ ionic state save
```

## Steps For Building The App
1. Firebase Configuration & Initialization
2. Create Firebase Service
3. Get Image from Camera
4. Convert Image to Blob
5. Save Image to Firebase Storage
6. Save Reference to Image in Firebase Realtime Database

### Firebase Configuration & Initialization

### Get Image From Gallery

### Convert Image to Blob
Firebase wants a blob to upload so we will follow the information in the Camera documentation for getting the file object and then read it into a blob. See the section in the Camera Plugin for additional information on the code below.

[Take A Picture And Get A File Entry Object](https://github.com/apache/cordova-plugin-camera#take-a-picture-and-get-a-fileentry-object-)



### Save Image to Firebase Storage

### Save Reference to Image in Firebase Realtime Database

### List Images