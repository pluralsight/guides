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




# Writing our first shaders

# Creating our first cube

# Rotating our cube

# Putting things into perspective


