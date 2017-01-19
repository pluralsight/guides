On December 2016, [React VR was pre-released](https://developer.oculus.com/blog/introducing-the-react-vr-pre-release/). It aims to allow web developers to author virtual reality (VR) applications using the declarative approach of [React](https://facebook.github.io/react/) and, in particular, [React Native](http://facebook.github.io/react-native).

React VR is built on top of [Three.js](https://github.com/mrdoob/three.js/) to offer support for the lower-level [WebVR](https://webvr.info/) and [WebGL](https://en.wikipedia.org/wiki/WebGL) APIs. WebVr is an API for accessing VR devices on the Web. WebGL (Web Graphics Library) is an API for rendering 3D graphics within any compatible web browser without the use of plug-ins.

React VR is similar to React Native because it uses `View`, `Image`, and `Text` as core components and supports Flexbox layouts. In addition, it adds VR components like `Pano`, `Mesh`, and `PointLight`. You can find more information on the [official site for React VR](https://facebookincubator.github.io/react-vr/docs/react-vroverview.html).

In this guide, we'll create a simple VR app to learn how to create a scene with a panorama image, 3d objects, buttons, and flexbox. It is based on two of the [official samples](https://github.com/facebookincubator/react-vr) of React VR, the Mesh and Layout samples.

The app will render 3d models of the Earth and Moon in a [cubemap](https://en.wikipedia.org/wiki/Cube_mapping) of the space along with some buttons to *zoom in* and *zoom out*. This is how the finished app looks like:

![finished app](https://raw.githubusercontent.com/pluralsight/guides/master/images/79aea4e7-a53b-4cfa-ba54-a03dc527c004.gif)

The models, their scale, and their rotation don't represent an accurate representation of the Earth or the Moon. This is just a simple example to demonstrate how React VR works. Along the way, we'll explain some concepts related to 3D modelling.

You can find the source code of the final app on [GitHub](https://github.com/eh3rrera/earth-moon-vr).

# Requirements

At this time of this writing, Virtual Reality is a new technology and we don't have that many choices to develop or test our VR apps.

The sites [WebVR](https://webvr.info/) and [Is WebVR Ready?](https://iswebvrready.org/) can help you know what browser and devices support the latest VR specs.

But don't worry, you don't need any special device ([Oculus Rift](https://www3.oculus.com/en-us/rift/), [HTC Vive](https://www.vive.com), [Samsung Gear VR](http://www.samsung.com/global/galaxy/gear-vr/)) to try a WebVR app right now.

Here's what you need:
- A Windows machine
- [Firefox Nightly](https://nightly.mozilla.org/) ([here are some instructions](https://github.com/Web-VR/iswebvrready/wiki/Instructions:-Firefox-Nightly))
- The latest version of [Node.js](https://nodejs.org/)

There are some [builds of Chrome](https://webvr.info/get-chrome/) that support the WebVR API, and even [an extension that emulates it](https://chrome.google.com/webstore/detail/webvr-api-emulation/gbdnpaebafagioggnhkacnaaahpiefil), but you'll get the best support with Firefox Nightly and Windows. 

Also, if you have an Android device and a Gear VR headset, you can install the [Carmel Developer Preview browser](https://www.oculus.com/experiences/gear-vr/1290985657630933/) to explore your React VR app with your headset.

# Creating the project

First, we need to install the React VR CLI tool:
```
npm install -g react-vr-cli
```

Using the React VR CLI, we can create a new application, let's say `EarthMoonVR`:
```
react-vr init EarthMoonVR
```

This will create an `EarthMoonVR` directory with a sample application inside, also installing the required dependencies so it can take a while (if you have [Yarn](https://yarnpkg.com/) installed, it will be used to speed things up).

Once it has finished, `cd` into that directory:
```
cd EarthMoonVR
```

To test the sample app, you can execute a local development server with:
```
npm start
```

Open your browser at http://localhost:8081/vr. It can take some time to build and initialize the app, but at the end, you should see the following:

![sample app](http://i.imgur.com/Ei3Qa7i.gif)

# React VR

The directory of this sample app has the following structure:
```
+-node_modules
+-static_assets
+-vr
\-.gitignore
\-.watchmanconfig
\-index.vr.js
\-package.json
\-postinstall.js
\-rn-cli-config.js
```

From all these files and directories, the ones we're going to work with are:

- `index.vr.js` that contains your application code.
- `static_assets` that contains the external resource files, like images and 3D models.

You can know more about the project structure [here](https://facebookincubator.github.io/react-vr/docs/project-configuration.html).

The following is the content of the `index.vr.js` file:
```javascript
import React from 'react';
import {
  AppRegistry,
  asset,
  StyleSheet,
  Pano,
  Text,
  View,
} from 'react-vr';

class EarthMoonVR extends React.Component {
  render() {
    return (
      <View>
        <Pano source={asset('chess-world.jpg')}/>
        <Text
          style={{
            backgroundColor:'blue',
            padding: 0.02,
            textAlign:'center',
            textAlignVertical:'center',
            fontSize: 0.8,
            layoutOrigin: [0.5, 0.5],
            transform: [{translate: [0, 0, -3]}],
          }}>
          hello
        </Text>
      </View>
    );
  }
};

AppRegistry.registerComponent('EarthMoonVR', () => EarthMoonVR);
```

We can see that React VR uses [ES2015](https://babeljs.io/learn-es2015/) and [JSX](https://facebook.github.io/react/docs/introducing-jsx.html).

The code is pre-processed by the [React Native packager](https://github.com/facebook/react-native/tree/master/packager), which provides compilation (ES2015, JSX), bundling, and asset loading among other things.

In the `return` statement of the `render` function, there's a:

- `View` component, which is typically used as a container for other components.
- `Pano` component, which renders a 360 photo (`chess-world.jpg`) that forms our world environment.
- `Text` component, which renders strings in a 3D space.

Notice how the `Text` component is styled with a style object. Every component in React VR has a `style` attribute that can be used to control its look and layout.

Aside from the addition of some special components like `Pano` or `VrButton`, React VR works with the same concepts of React and React Native, such as components, props, state, lifecycle methods, events, and flexbox layouts.

Finally, the root component should register itself with `AppRegistry.registerComponent` so the app can be bundled and run.

With a better understanding of what this code does, let's dive into panorama images.

# Pano images

Generally, the world inside our VR app is composed by a panorama (pano) image, which creates a sphere of 1000 meters (in React VR distance and dimensional units are in meters) and placing the user at its center.

A pano image allows you to see the image from every angle including above, below, behind and next to you, that's the reason they are also called 360 images or spherical panoramas.

There are two main formats of 360 panoramas, equirectangular and cubic. React VR supports both.

An equirectangular pano consists of a single image with an aspect ratio of 2:1, meaning that the width must be twice the height.

These images are created with a special 360 camera. An excellent source of equirectangular images is [Flickr](https://www.flickr.com), you just need to search for the *equirectangular* tag. For example, by trying [this search](https://www.flickr.com/search/?text=equirectangular&license=2%2C3%2C4%2C5%2C6%2C9), I found [this  photo](https://www.flickr.com/photos/54144402@N03/9581334556/):

![equirectangular example](https://farm4.staticflickr.com/3804/9581334556_fb0cfafa19_z_d.jpg)

Looks weird, doesn't it?

Anyway, download the photo (at the highest resolution available) to the `static_assets` directory of our project and change the `render` function to show it:
```javascript
render() {
    return (
      <View>
        <Pano source={asset('sample_pano.jpg')}/>
      </View>
    );
  }
```

The `source` attribute of the `Pano` component takes an object with an `uri` property with the location of the image. Here we are using the `asset` function that will look for the `sample_pano.jpg` file in the `static_assets` directory and return and object with the correct path in the `uri` property. In other words, the above code is equivalent to:
```javascript
render() {
  return (
    <View>
      <Pano source={ {uri:'../static_assets/sample_pano.jpg'} }/>
    </View>
  );
}
```

When we refresh the page in the browser (and assuming the local server is still running), we should see something like this:

![sample pano](http://i.imgur.com/Zx4qKcD.gif)

By the way, if we want to avoid refreshing the page at every change, we can enable hot reloading by appending `?hotreload` to the URL (http://localhost:8081/vr/?hotreload).

Cubemaps are the other format of 360 panoramas. This format uses six images for the six faces of a cube that will fill the *sphere* around us. It's also known as a skybox.

The basic idea is to render a cube and place the viewers at the center, following them as they move.

For example, consider this image thata represent the sides of a cube:

![cube image](https://raw.githubusercontent.com/pluralsight/guides/master/images/65d27e9a-007c-4618-aa92-79165e074167.png)
