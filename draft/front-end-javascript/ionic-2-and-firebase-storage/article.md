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

### Create Firebase Service

### Get Image From Gallery

### Convert Image to Blob
Firebase wants a blob to upload so we will follow the information in the Camera documentation for getting the file object and then read it into a blob. See the section in the Camera Plugin for additional information on the code below.

[Take A Picture And Get A File Entry Object](https://github.com/apache/cordova-plugin-camera#take-a-picture-and-get-a-fileentry-object-)

```javascript
 window.resolveLocalFileSystemURL(imageData, (fileEntry) => {

    fileEntry.file((resFile) => {

      var reader = new FileReader();
      reader.onloadend = (evt: any) => {

        var imgBlob: any = new Blob([evt.target.result], { type: 'image/jpeg' });
        imgBlob.name = 'sample.jpg';
        
        // Will complete this function an upcoming section
        FirebaseService.uploadPhotoFromFile(imgBlob)

      };
      reader.onerror = (e) => {
        console.log("Failed file read: " + e.toString());
      };
      reader.readAsArrayBuffer(resFile);

    });
  }, (err) => {
    console.log(err);
    alert(JSON.stringify(err))
  });
```

### Save Image to Firebase Storage

We are going to add the code below to the `FirebaseService.ts` file we created earlier.

```javascript
uploadPhotoFromFile(_imageData) {

    return new Observable(observer => {
    
        // create a name for the file using the current time
        var _time = new Date().getTime()
        var _fileName = 'images/sample-' + _time + '.jpg';
        var fileRef = firebase.storage().ref(_fileName)
        var uploadTask = fileRef.put(_imageData);

        // track here the progress of the file upload
        uploadTask.on('state_changed', function (snapshot) {
            console.log('state_changed: file upload info', snapshot);
            
        }, function (error) {
            // if we got an error, return the information
            console.log(JSON.stringify(error));
            observer.error(error)
            
        }, function () {
            // Handle successful uploads on complete

                // completed in next section
                saveImageReference(_uploadTask, _fileName)
                
                observer.next(uploadTask)
            }).catch(function (error) {
                // An error occurred!
                console.log(JSON.stringify(error));                
                observer.error(error)
            });

        });
    });
}
```    

### Save Reference to Image in Firebase Realtime Database

Since you cannot query the data storage directly, we need to keep information regarding the upload images in the Firebase Realtime Database also. The code below will save basic information regardin the image in a new reference path `images`.

Add the code below to the `FirebaseService.ts` file we created earlier.

```javascript
function saveImageReference(_uploadTask, _fileName) {
    // save a reference to the image for listing purposes,
    // in the realtime database
    var ref = firebase.database().ref('images');
    ref.push({
        'imageURL': _uploadTask.snapshot.downloadURL,
        'fileName': _fileName,
        'when': new Date().getTime(),
    });
}
```

### List Images