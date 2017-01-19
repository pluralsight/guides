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

This will create an `EarthMoonVR` directory with a sample application inside and install the required dependencies, so it can take a while (if you have [Yarn](https://yarnpkg.com/) installed, it will be used to speed things up).

Once it has finished, `cd` into that directory:
```
cd EarthMoonVR
```

To test the sample app, you can execute a local development server with:
```
npm start
```

Now open your browser at http://localhost:8081/vr. It can take some time to build and initialize the app, but at the end, you should see the following: