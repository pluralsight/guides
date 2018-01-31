# What is Augmented Reality?
Augmented reality is simply the ability to digitally place virtual elements into the real world and interact with these elements as if they were actually present.  AR opens exciting new possibilities in many areas, including navigation, gaming, military technology, travel enhancements, and more. 

We can spruce up the real world environment in ways impossible before AR. The possibilities are endless, limited only by our creativity.

Apple announced their **ARKit framework** at World Wide Developers Conference (WWDC) in June 2017. The ARKit framework relies on advanced iOS features like real-time world tracking and hit testing, horizontal plane detection, and the ability to apply real lighting to virtual objects. 

But we don’t have to be experts in all these areas. ARKit hides all the underlying complexity and makes creating AR applications rather simple. 

Here’s a screenshot from the AR-demo we’re going to build in this tutorial: 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/66eb6c0f-bb81-4937-b477-7e968e153310.PNG)

Follow along with me to see how we get there. You can download the demo projects from [Github](https://github.com/nyisztor/arkit-intro)

# Your First AR App

You're going to create your first iOS augmented reality app in a matter of minutes. Fired up? Read along.

## Prerequisites

To implement the ARKit demos in the tutorial, you’ll need a **Mac** with **Xcode 9** or newer. 
Also, ARKit requires the A9 or newer processors to perform all the complex computations in real-time. Thus, you’ll need an **iPhone SE, iPhone 6s, iPhone 6s Plus, 5th generation iPad or a newer device running iOS 11**. 

**Note that ARKit apps won’t run in the Simulator.**

## Let’s get started

Apple made it easy for us to create beautiful, realistic augmented reality apps and games through Xcode 9 and the hardware described above. Xcode 9 even comes with a dedicated template that allows us to build our first ARKit app within minutes. 

Open up Xcode and choose *File -> New -> Project*. Choose the *Augmented Reality App* template:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd9b2731-52b3-4146-99e9-fff4ba911cfd.png)

Hit *Next*, then pick a name for your project (e.g., “FirstARKitApp”). You can leave the defaults for the other settings (*Language: Swift, Content Technology: SceneKit*). Click *Next* then create the project in a folder of your choice.

> Congrats! You just created your very first augmented reality app.  

Yes, you read that right! **No coding required.** And here’s what you’ll see after running the app on a real device:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0209a2ef-48b3-4ad1-81ef-29232b82b63e.PNG)

Now, let’s take a closer look at the generated project. The *Main.storyboard* contains a single view controller which has a main view of type ```ARSCNView```. ```ARSCNView``` is a special view used for displaying scenes that blend the device’s camera view with virtual 3D content. The view is connected to the sceneView outlet in the ```ViewController.swift``` file:  
```
@IBOutlet var sceneView: ARSCNView!
```
Next, let’s inspect the ViewController’s ```viewDidLoad()``` method: 
```
override func viewDidLoad() {         
    super.viewDidLoad()          
    
    // Set the view's delegate         
    sceneView.delegate = self        
    
    // Show statistics such as fps and timing information         
    sceneView.showsStatistics = true     
    
    // Create a new scene         
    let scene = SCNScene(named: "art.scnassets/ship.scn")!  
    
    // Set the scene to the view         
    sceneView.scene = scene     
}
```

The sceneView’s delegate is set to `self`, but we won’t be using the ```ARSCNViewDelegate``` in this demo. 
```
sceneView.showsStatistics = true  
```
By setting the `showsStatistics` attribute to true, we’ll see real-time statistics when running our app. 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6773258a-c6c9-416c-b70a-88c2970bc87a.jpg)

Tapping the `+` icon in at bottom left reveals even more details:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3a28d02f-29bf-448b-a706-21b402695104.jpg)

- The green bar is the performance indicator. A full bar indicates good performance.
- The rendering engine is displayed to its right. Possible values are GL (OpenGL) and Mt (Metal). On modern devices capable of running ARKit you’ll see Mt.
- The FPS counter shows the frame rate, or the number of screen updates per second. 60 is the maximum and values above 30 are acceptable.
- The node count (the value next to the diamond-shaped icon) shows the number of nodes in your scene graph. More on this later.
- The next piece of information is the polygon count indicator. It shows how many polygons are rendered currently in your scene.
- The donut chart shows what the time is being spent on while rendering a frame. We can also see how much time it takes to render a frame.  
In this example, it only took 0.4 ms to display a frame. Thus, we could display up to 2500 frames/s if the device supported such a high frame-rate.   

We load the scene next: 
```
let scene = SCNScene(named: "art.scnassets/ship.scn")!  
```
An ```SCNScene``` instance represents a 3D scene and its contents, including lights, 3D-objects, cameras and other attributes. The current scene is initialized using the *ship.scn* that is bundled with the ARKit template:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/70042726-21dc-4c44-adc9-b2a298047862.png)

This ship will be displayed and combined with the real-world in our demo.
Finally, we assign the scene to the ```ARSceneView``` object:
```
sceneView.scene = scene  
```
We’re almost ready to run our first AR-app. The final configuration steps are performed in the ```viewWillAppear()``` delegate method:
```
override func viewWillAppear(_ animated: Bool) {         
    super.viewWillAppear(animated)        
    // Create a session configuration         
    let configuration = ARWorldTrackingConfiguration()
    // Run the view's session         
    sceneView.session.run(configuration)
}
```
An ```ARSCNView``` has an AR session associated with it. The ```ARSession``` processes the camera image and tracks the device’s motion. The AR-processing starts after calling the ```ARsession```’s ```run()``` method. The ```run()``` method needs a valid ```ARConfiguration``` instance. This configuration object defines the session’s motion and scene tracking behaviors.  

```ARConfiguration``` cannot be instantiated directly. Instead, use one of the following subclasses provided by ARKit. 

*Quoting from the ARKit documentation*:
- ```ARWorldTrackingConfiguration``` Provides high-quality AR experiences that use the rear-facing camera precisely track a device's position and orientation and allow plane detection and hit testing.
- ```AROrientationTrackingConfiguration``` Provides basic AR experiences that use the rear-facing camera and track only a device's orientation.
- ```ARFaceTrackingConfiguration``` Provides AR experiences that use the front-facing camera and track the movement and expressions of the user's face. 

The ARKit template uses the ```ARWorldTrackingConfiguration``` class. This type of configuration will track the device’s position and orientation changes as well. As a result, we’ll be able to freely move around the spaceship model which stays at a fixed position in the real world place.

If I instead used an ```AROrientationTrackingConfiguration```, the device’s position changes would not be tracked. As a consequence, the starship will move together with the device. We could use this configuration to display the planets of the solar system orbiting around our device.

However, the lack of position change tracking is a deal-breaker for most AR apps and games, so you’ll probably end up using ```ARWorldTrackingConfiguration```.  In addition to position and orientation changes, ```ARFaceTrackingConfiguration``` can also track *facial expressions*, which opens further opportunities. 

> Note that ARFaceTrackingConfiguration requires the iPhoneX’s TrueDepth camera.

# Detecting Planes

To successfully blend virtual scenes with the real, physical world, we need to be able to identify the geometry of our surroundings. ARKit can detect flat surfaces in the real world and provides their position and orientation. This information lets us place virtual objects on ceilings, tables or other horizontal planes in the real world. 

I created a clone of the FirstARKitApp demo we used previously and named it PlaneDetection. We won’t need any scene assets in this demo, so I deleted it. I also removed the 3D-scene initialization code from the ```ViewController```’s ```viewDidLoad()``` method, which should only contain the following lines of code:
```
override func viewDidLoad() {         
    super.viewDidLoad()        
    
    // Set the view's delegate
    sceneView.delegate = self
    
    // Show statistics such as fps and timing information
    sceneView.showsStatistics = true    
}
```
 Next, we need *enable plane detection* in ARKit. This can be achieved by setting the ```planeDetection``` property on the session configuration object to ```ARWorldTrackingConfiguration.PlaneDetection.horizontal``` in the ```viewWillAppear()``` delegate method:  
```
 override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)        
    
    // Create a session configuration
    let configuration = ARWorldTrackingConfiguration()
    
    // Enable horizontal plane detection
    configuration.planeDetection = .horizontal
    
    // Run the view's session
    sceneView.session.run(configuration)     
}
```

With this setting in place, the session will detect horizontal, flat surfaces in the real world geometry captured by the device’s front side camera.

> To disable plane detection set the planeDetection property to an empty array literal.

To visualize how plane detection works, we need to add the ```ARSCNDebugOptions.showFeaturePoints``` value to the scene view’s ```debugOptions```.
```
override func viewDidLoad() {
    super.viewDidLoad()        
    …
    sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints]        
    …
}
```

Running the app will show visual indicators, the so-called feature points as ARKit finds continues surfaces in real-time:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8e8b9858-7413-47a2-b7ed-417cbb9d97ac.PNG)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9a48d9eb-9a8a-4ea4-a582-c884aef130a6.PNG)

That’s cool, but how about visualizing the detected planes?

Here’s where we need the ```ARSCNViewDelegate```. As you may recall, the ```ARSCNView```’s delegate is set to self in the ```viewDidLoad()``` delegate method:  
```
override func viewDidLoad() {         
    super.viewDidLoad()
    
    // Set the view's delegate
    sceneView.delegate = self        
    …     
}
```
The ```ARSCNViewDelegate``` protocol provides useful delegate methods. In this demo, we’ll rely on the ```renderer(_:didAdd node: for anchor:)``` delegate method. ARKit calls this delegate method whenever a new node corresponding to an anchor has been added to the scene. 

Let’s take a closer look at the method: 
```
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor)  
```
The first parameter is a scene renderer. This is our ```ARSCNView``` instance.  
The second one is a ```SCNNode``` object. The third parameter is an AR anchor object corresponding to the node.  

So, what *is* an AR anchor object? **AR anchors are used for tracking the real-world positions and orientations of real or simulated objects relative to the camera.**  When we enable horizontal plane detection, ARKit calls the ```renderer(_: didAdd node:, for anchor:)``` delegate method automatically whenever it detects a new horizontal plane and adds a new node for it. We receive the anchor of each detected flat surface, which will be of type ```ARPlaneAnchor```. 

```ARPlaneAnchor``` represents a planar surface in the world, defined using X and Z coordinates, where Y is the plane’s normal.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/db7b85f6-e399-487d-b7f8-a319fce6c205.png)


Let’s implement the ```renderer(_: didAdd node:, for anchor:)``` method:  
```
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
    if let planeAnchor = anchor as? ARPlaneAnchor {
        let plane = SCNPlane(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.z))  plane.firstMaterial?.diffuse.contents = UIColor(white: 1, alpha: 0.75)
        
        let planeNode = SCNNode(geometry: plane)
        
        planeNode.position = SCNVector3Make(planeAnchor.center.x, planeAnchor.center.x, planeAnchor.center.z)         
        planeNode.eulerAngles.x = -.pi / 2          
        
        node.addChildNode(planeNode)     
    } 
}
```        
Here’s a detailed explanation of what this code does:      
```
if let planeAnchor = anchor as? ARPlaneAnchor 
{           
    …     
}
```
We process only anchors of type ```ARPlaneAnchor```, since we’re only interested in detecting planes.     
```
let plane = SCNPlane(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.z))
```
To visualize the detected planes, I create an ```SCNPlane``` object. The ```SCNPlane``` is a SceneKit type representing a one-sided plane geometry. 

Next, I set the plane’s ```width``` and ```height``` to the anchor ```extent```’s x and y values. The ```ARPlaneAnchor``` object’s ```extent``` property provides the estimated size of the detected plane. So, the ```SCNPlane``` object will have the same size as the detected plane in the world.
```
   plane.firstMaterial?.diffuse.contents = UIColor(white: 1, alpha: 0.75)
```
Next, I set the plane’s diffuse material to white with 75% transparency.
```
   let planeNode = SCNNode(geometry: plane)         
   planeNode.position = SCNVector3Make(planeAnchor.center.x, planeAnchor.center.y, planeAnchor.center.z) 
   planeNode.eulerAngles.x = -.pi / 2
```     
A ```SCNode``` gets created with the plane geometry. The node’s position will match the anchor’s position to give us an accurate visual clue. I use the anchor ```center```’s coordinate’s to create a 3D vector. ```SCNPlane``` is one-sided and would appear perpendicular to the surface by default. I need to rotate the ```planeNode``` 90 degrees counter-clockwise to make it display correctly.
```
   node.addChildNode(planeNode)
```
Finally, we make the planeNode a child node of the newly created scene node.

Running the app will produce results like this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a3ede949-b37e-4767-bfc3-b3cec7bdc453.jpg)

ARKit keeps monitoring the environment and updates the previously detected anchors. We can subscribe to these updates by implementing the ```renderer(_:, didUpdate node:, for anchor:)``` ARSCNViewDelegate delegate method. 

Let’s implement the delegate so that we can display more accurate results.
```
func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
    if let planeAnchor = anchor as? ARPlaneAnchor,         
    let planeNode = node.childNodes.first,         
    let plane = planeNode.geometry as? SCNPlane {
        plane.width = CGFloat(planeAnchor.extent.x)         
        plane.height = CGFloat(planeAnchor.extent.z)
        planeNode.position = SCNVector3Make(planeAnchor.center.x, 0, planeAnchor.center.z)     
    } 
}
```
First, we perform some safety checks to ensure that we’re working with plane information. Then, we set the ```SCNPlane``` object’s ```width``` and ```height```. Finally, we update the child node’s position.

Now, we are able to detect larger continuous surfaces and display the plane overlays more accurately.

Build and run the app on a device. Allow ARKit to collect enough information about the environment by moving your device around flat surfaces. You’ll see ARKit in action as it updates the size and the position of the detected surfaces.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b6f463f4-f35c-4f30-a82e-507096401c6f.jpg)

# Virtual 3D Shapes in Your Living Room

How about placing some 3D-objects in our living room? Now that we can detect those flat surfaces, it’s relatively easy to build a scene which blends reality with virtual objects.

We’ll continue enhancing the app from where we left off. First, we need a ```SceneKit``` scene. Right click on your project in the project navigator, and choose “New File…” Pick “SceneKit Scene File” and click *Next*. I saved it as *scene3d.scn*.

We can use the *SceneKit Editor* to place various pre-defined 3D-shapes to our scene: boxes, spheres, pyramids and so on. We could also create these shapes in code, but I used the scene editor to keep it brief.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4da60cae-b50e-4b74-9b60-f9ce40c12897.png)

I created a private method to load the scene:
```
private func loadScene() {
    guard let scene = SCNScene(named: "art.scnassets/geometry.scn") else {
        print("Could not load scene!")             
        return         
    }
    
    let childNodes = scene.rootNode.childNodes        
    for childNode in childNodes {
        sceneNode.addChildNode(childNode)         
    }     
}
```
I also added a property called ```sceneNode```:  
```
var sceneNode = SCNNode() 
```
This will be the parent node for the nodes in the scene.  I call the ```loadScene()``` method from ```viewDidLoad()```, but I don’t want to display the scene yet. I aim to place the scene once ARKit detects a horizontal plane. 

As you may recall, the ```renderer(_:, didAdd node:, for anchor:)``` delegate method gets called whenever a new anchor is detected, and a corresponding node is created. So, all we have to do is to add our scene to this node. Here’s our enhanced delegate method:
```
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
    if let planeAnchor = anchor as? ARPlaneAnchor {        
        …              
        if !isSceneRendered {                 
            isSceneRendered = true
            sceneNode.scale = SCNVector3(0.01, 0.01, 0.01)                 
            sceneNode.position = SCNVector3Zero                 
            node.addChildNode(sceneNode)             
        }     
    } 
}
```    
Note that I created a new boolean property. By checking the ```isSceneRendered``` property, we ensure that the scene is only added to the node belonging to the earliest detected horizontal plane. 

The other couple of lines of code set the ```scale``` and the ```position``` of the scene node. 

Finally, I add the ```sceneNode``` as a child node.

Now walk around your home with the app running on your iPhone. ARKit will soon detect a flat surface, and you should see the scene appearing at random places:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0036d9ff-6076-4e18-b9c7-c76e6b9dc1ef.PNG)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf003f0e-cf95-443b-90d0-ee80123a20fe.PNG)

Devices supporting ARKit can render more complex scenes. Here’s a scene I put together using some free 3D models available on [free3d.com](www.free3d.com)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2b87593c-178e-46e6-be2d-f16cf81719bf.PNG)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9e15f7ec-c880-4a50-9580-54ae062dac4a.PNG)

# Where to Go from Here?

Hopefully, you can see some potential in AR and have seen some of the neat features of ARKit. If you got a little lost in the code, don't fret. Check out the [demo projects on Github](https://github.com/nyisztor/arkit-intro)!

If you enjoyed this quick introduction to ARKit, you'd probably want to find out more. 

I collected a couple of links for you: 
- Introducing ARKit (WWDC 2017): https://developer.apple.com/videos/play/wwdc2017/602/ 
- Official ARKit site: https://developer.apple.com/arkit  
- Free and premium 3D models for your ARKit/SceneKit apps: https://free3d.com, https://sketchfab.com, https://www.turbosquid.com 
- Blender, free 3D creation tool: https://www.blender.org

Also, you may want to check out my [Swift courses on Pluralsight](https://www.pluralsight.com/authors/karoly-nyisztor). I talk about design patterns and Swift best-practices, tips and tricks.

I hope you found this guide informative and engaging. Please **leave your comments and feedback** in the discussion section below, and definitely hit the "favorites" button!  

Thanks for reading!

Cheers,
Károly