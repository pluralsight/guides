# What is WebGL?

[WebGL](https://www.khronos.org/webgl/) enables plugin-free 3D content on the web.
That means it is an interface between JavaScript and the OpenGL API, which is designed for high-performance 2D and 3D graphics.

# Setting up our environment

WebGL is using the [Canvas element](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement) and its `getContext` method to provide access to the WebGL functionality, so as the very first thing, we'll need an HTML file with a canvas element inside.

Create a new file called `index.html` with the following content:
```html
<!doctype html>
<html>
  <head>
    <title>WebGL 101</title>
  </head>
  <body>
    <canvas width="500" height="500"></canvas>
    <script id="vertexShader" type="text/plain">
      /* We will put the vertex shader code here */
    </script>

    <script id="fragmentShader" type="text/plain">
      /* We will put the fragment shader code here */
    </script>
    
    <script src="main.js"></script>
  </body>
</html>
```

* The `<canvas>` element will later hold our graphics.
* The two scripts with type `text/plain` are not executed by the browser. We will use them to hold the source code of two programs we need for rendering later, the so-called "shaders".

Now in `main.js` it is time to get access to WebGL for the canvas we have created and set us up for rendering with a loop that does the drawing later:

```javascript
var canvas = document.querySelector(canvas)
var gl = canvas.getContext('webgl')

/*
All the setup will go here
*/

// Ask the browser when it's ready to draw something
requestAnimationFrame(function render() {
  // the drawing happens here
  requestAnimationFrame(render)
})
```

# WebGL lingo & pipeline

Before start writing the code needed to get some visuals rendered, we should have a look at what parts we need to get going.

We already have a Canvas to draw on and we've got a "render loop" where we can put pixels onto that canvas as fast as the browser can keep up.

But we will need a few more things in order to see something appear on screen with WebGL - in fact there is a pipeline involved that takes data on what to draw, processes this data with programs (the "shaders" mentioned earlier) and then draws the pixels according to the transformed data.

Let's see what that pipeline looks like:

![The WebGL pipeline from data to pixels](https://raw.githubusercontent.com/pluralsight/guides/master/images/48392913-d9df-4f3d-ace0-43e9699b44cb.png)

The colours are indicating that there's three big differences between the boxes:

* **Blue** means this is JavaScript code.
* **Green* means a step of the pipeline that happens on the [GPU](https://en.wikipedia.org/wiki/GPU).
* **Red** means it is a program that will run on the GPU and is written in [GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language).

The image also uses a few words that you may not be familiar with, so here's what they mean:

* A **Vertex** is simply a point we will use. To be precise, a vertex is a point that connects two line segments.
* A **Buffer** is basically an array, but it is managed by WebGL and, if possible, will reside in graphics memory.
* The **Vertex Shader** is a program that we will write later. It allows us to manipulate the vertices very efficiently before they get drawn. Here we can do things like scaling, rotation, perspective transformation. It get a vertex as input and our code has to return the vertex we want to be used for the rest of the pipeline. We will have a look at this in detail later.
* The **Rasterizer** is a component that figures out what pixels are inside the shapes we are drawing and then passes these pixels into the next step of the pipeline
* The **Fragment Shader** is another program that we'll write and instead of a vertex it gets the position of a fragment pixel, i.e. one pixel that the rasterizer found to be inside our shapes. Our program will then figure out what colour we want to use when we put that pixel on the screen. We will also have a closer look at this later.

With all that out of the way, we are ready to go forward and put some pixels on the canvas!

# Creating our first triangle

Following our pipeline, we will have to create a list of vertices we want to draw.
The coordinate system, by default, is slightly different than what you may expect:


![The coordinate system for WebGL ranges from -1.0 to 1.0 on each axis, making (0,0) the center of the screen](https://raw.githubusercontent.com/pluralsight/guides/master/images/7d46f90e-c5a1-4b35-aa92-414a15e54d09.png)


Especially when you worked with [the Canvas 2D](https://developer.mozilla.org/en/docs/Web/API/CanvasRenderingContext2D) API before, this coordinate system will be unfamiliar for you.

So the center of the screen is at position `x=0.0, y=0.0`, while the bottom left-hand corner is at `x=-1.0, y=-1.0` and the top right-hand corner is at `x=1.0, y=1.0`.

So we will be dealing with float values for coordinates.
Let's figure out three points for our triangle:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7f55b8f6-45c1-44a4-af29-6bb38808e8a4.png)

So we have three points:

* x = -0.5, y = -0.5
* x =  0.0, y =  0.5
* x =  0.5, y = -0.5

We can now put these three points into an array. JavaScript arrays aren't optimal for WebGL, because WebGL makes a few assumptions on the way the data is put into memory, so we will convert this array into a [Typed Array](https://developer.mozilla.org/en/docs/Web/JavaScript/Typed_arrays) for 32 Bit Float numbers, a [Float32Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float32Array):

```javascript
var points = [
  -0.5, -0.5,
   0.0,  0.5,
   0.5, -0.5
]

var vertices = new Float32Array(points)
```

With this typed array, we can create a buffer for our vertices and put the content of the typed array into the buffer:

```javascript
var vertexPosBuffer = gl.createBuffer()

// WebGL uses the concept of the "bound buffer" with a few predefined buffer "targets". Each target can only be assigned a single buffer at the same time. We make our vertexPosBuffer the assigned buffer for the ARRAY_BUFFER target
gl.bindBuffer(gl.ARRAY_BUFFER, vertexPosBuffer)
// now we can set the data for the ARRAY_BUFFER target to our typed array. We also specify how the data is going to be used. STATIC_DRAW means: The data is written once and used many times (each time we draw)
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW)
```

There were a few new things in the last code snippet, let's have a look at them in more detail.

First of all, WebGL has a few predefined buffers, called "targets":

* `ARRAY_BUFFER` for vertex attributes. That means: Data that is related to each vertex should be put here when used in the pipeline
* `ELEMENT_ARRAY_BUFFER` for indices. That means: If you refer to a certain index in the `ARRAY_BUFFER`, you can specify it here

**Note**: There are more such targets for [WebGL 2](https://developer.mozilla.org/en-US/docs/Web/API/WebGL2RenderingContext) listed [here](https://developer.mozilla.org/en/docs/Web/API/WebGLRenderingContext/bufferData).

Once we have our buffer bound to a target, we can use that target to put data into our buffer by using `bufferData` with the target and the typed array with data. There is also a third parameter giving WebGL a hint on how we will use the information:

* `STATIC_DRAW` is meant for data that is not changed but going to be used often
* `DYNAMIC_DRAW` is meant for data that is going to be changed and used often
* `STREAM_DRAW` is meant for data that is going to be used only a few times 

**Note**: There are more such usage hints for [WebGL 2](https://developer.mozilla.org/en-US/docs/Web/API/WebGL2RenderingContext) listed [here](https://developer.mozilla.org/en/docs/Web/API/WebGLRenderingContext/bufferData).

As we are putting our vertices in once and then use them for the rest of the time the application is running to draw the triangle, we will stick with `STATIC_DRAW`.

Now we have the data ready for the next steps of the pipeline: Shaders.

We will have to create two shaders, the *Vertex Shader* and the *Fragment Shader* and link them together into a *Program* that can be used to calculate the final vertex positions and colours to draw on screen.

We will start by using some very simple shaders and put their source code into the two `<script>` tags we prepared at the beginning of the tutorial:

```html
    <script id="vertexShader" type="text/plain">
      vec4 aVertexPos; // the input goes here: A single vertex (x,y, 0, 0) from the buffer
      
      void main(void) {
        gl_Position = aVertexPos; // we just output the original vertex position
      }
    </script>
    <script id="fragmentShader" type="text/plain">
      void main(void) {
        // we will fill the triangle with red (RGBA = 1.0, 0.0, 0.0, 1.0)
        gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
      }
    </script>
```

# Writing our first real shaders

# Creating our first cube

# Rotating our cube

# Putting things into perspective


