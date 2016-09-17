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

We will dig deeper into shaders in the next section, but for now here's the most important bits we need to understand:

* Shaders are written in a programming language called [GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)
* We will put our vertices one by one into `aVertexPos`
* The GPU will run the vertex shader once for each of the vertices, producing the vertices it will use for the rest of the pipeline
* The fragment shader will make every pixel in our triangle appear red

This is just enough for us to be able to continue in our quest to get a triangle on screen.

As said, we need to compile the shaders and link them together into a program, so we will add some more JavaScript code:

```javascript
// get the source code from the two script tags
var vertexShaderSrc = document.getElementById("vertexShader").value
var fragmentShaderSrc = document.getElementById("fragmentShader").value

// create the program and shaders
var shaderProgram = gl.createProgram()
var vs = gl.createShader(gl.VERTEX_SHADER)
var fs = gl.createShader(gl.FRAGMENT_SHADER)

// compile the vertex shader
gl.shaderSource(vs, vertexShaderSrc)
gl.compileShader(vs)
if(!gl.getShaderParameter(vs, gl.COMPILE_STATUS)) {
  alert('Vertex shader failed compilation:\n' + gl.getShaderInfoLog(vs))
  console.log(gl.getShaderInfoLog(shaderProgram))
}

// compile the fragment shader
gl.shaderSource(fs, fragmentShaderSrc)
gl.compileShader(fs)
if(!gl.getShaderParameter(fs, gl.COMPILE_STATUS)) {
  alert('Fragment shader failed compilation:\n' + gl.getShaderInfoLog(fs))
  console.log(gl.getShaderInfoLog(shaderProgram))
}

// put the shader binaries into the program & link the program
gl.attachShader(shaderProgram, vs)
gl.attachShader(shaderProgram, fs)
gl.linkProgram(shaderProgram)
if(!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
  alert("Could not initialise shaders")
}

// make the program active to be used from now on
gl.useProgram(shaderProgram)
```

With this code we take the shader sourcecode from the script tags, compile them into binaries for the GPU and put them together as a "program" that we then use for the rest of the work.

We are finally homing in on target! There is only two things left: We need to point the variable (in a vertex shader this is called an `attribute`, hence the prefix `a` in the name `aVertexPos`) to our buffer data. To do this, we get the attribute number represented by `aVertexPos` in the code of the program, enable the array for this attribute and then point the attribute to our current `ARRAY_BUFFER` which contains the vertex positions:

```javascript
// gets the index for aVertexPos which is needed for enabling the attribute & pointing it to data
var attribIndex = gl.getAttribLocation(shaderProgram, 'aVertexPos') 
gl.enableVertexAttribArray(attribIndex)
/* 
  Now we tell the attribute how the data is going to be used:
  - numbers per vertex (x & y coordinates)
  - each number is a FLOAT
  - because it is already FLOAT, we do not need to convert
  - there is no values between the values for the vertices
  - we begin at the first value in the buffer
*/
gl.vertexAttribPointer(attribIndex, 2, gl.FLOAT, false, 0, 0)
```

Now we can finally draw it out:

```javascript
gl.drawArrays(gl.TRIANGLES, 0, 3)
```
This draws a triangle from the first vertex (number 0) and uses 3 vertices.
Here is our result:

![The red triangle we created](https://raw.githubusercontent.com/pluralsight/guides/master/images/f2898b24-454e-418d-9f3f-042b8be6b2fd.com_2016-08-04_12-32-09)

# Writing our first real shaders

Previously we have skipped over shaders and just wrote the most trivial shaders needed to get some colour on screen.

Now, what *exactly* are shaders, anyway?

Shaders are programs, written in the [GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language) programming language that run on the graphics processor (GPU).

These programs allow us to define what we will draw on screen without having to resort to JavaScript and change the entire data in memory. If you look at the amount of data we may have to work with, this is great news!

For instance, for a 3D game we may have to deal not just with three vertices for a triangle, but with a million vertices for a game world, player and enemies.

Shaders have a few advantages here:

* as they run on the GPU, they run on specialised hardware that is optimised for handling graphics data
* they can run in parallel, processing many vertices or pixels at the same time
* they are written in a programming language that has many useful helpers to work with coordinates, colours etc.

So for example, if we want to make our triangle larger or smaller, rather than changing the vertices in our buffer, we can use the **vertex shader** to move the vertices closer together or farther apart:

```glsl
vec4 aVertexPos; // the input goes here: A single vertex (x,y, 0, 0) from the buffer
  
void main(void) {
  vec4 newPosition = aVertexPos;
  newPosition.x *= 2.0;
  newPosition.y *= 2.0;

  gl_Position = newPosition;
}
```

Here we create a new local variable in the vertex shader called `newPosition` and assign the values from `aVertexPos`.
We then multiply the `x`- and `y`-coordinate with two. When this shader is run on each vertex, it will make the vertices twice as far apart from each other, as demonstrated here:

![The scaled vs. the original triangle](https://raw.githubusercontent.com/pluralsight/guides/master/images/773993a4-1280-4794-a82f-6c3c3019944f.png)

In red are our vertices as they are in the buffer, in green are the vertices as they are returned by the vertex shader.

There are other ways to write the vertex shader from the last example with less code, leveraging the features GLSL gives us for free:

```glsl
vec4 aVertexPos; // the input goes here: A single vertex (x,y, 0, 0) from the buffer
  
void main(void) {
  // vec4(...) creates a new array with four items, called a "four component vector"
  // we can use ".x", ".y" and so on to access a single item from such an array
  // we can also use ".xy" to get an array with the first two items from the array
  // vec4 can be used to create a new array with smaller arrays and additional values
  // GLSL allows us to multiply an array with a number, e.g. [1,2] * 2 = [2, 4]
  gl_Position = vec4(newPosition.xy * 2.0, 1.0, 1.0);
}
```
Remember: The code is identical to the longer code we've used before, but makes use of many helpful features from GLSL. Here's a slightly longer break down:

**Vectors:**
In Geometry, we like to call an array that tells us the direction from somewhere to somewhere else a *vector*. For instance `[1,2]` can be used to say "1 pixel to the right, 2 pixels to the top", which means the first item in the array means how much we travel to the right on the x-axis and the second means how much we travel up on the y-axis.

In GLSL we would write this as `vec2(1.0, 2.0)`:

```glsl
vec2 moveIt = vec2(1.0, 2.0); // moveIt.x == 1.0, moveIt.y == 2.0
```

We can create vectors with different amounts of components (that's how items in vectors are called):

```glsl
vec2 a = vec2(1.0, 2.0);
vec3 b = vec3(1.0, 2.0, 3.0);
vec4 c = vec4(1.0, 2.0, 3.0, 4.0);
```

We will learn later, that vectors can be used to express different things and it's good to have 2-, 3- and 4-component vectors.

GLSL also gives us a bunch of convenience functions to create vectors from larger or smaller vectors:

```glsl
vec2 a = vec2(1.0, 2.0);
vec3 b = vec3(1.0, 2.0, 3.0);
vec4 c = vec4(1.0, 2.0, 3.0, 4.0);

vec3 b2 = vec3(a, 3.0); // the same result as "b"
vec4 c2 = vec4(a, a);   // results in vec4(1.0, 2.0, 1.0, 2.0)
vec2 a2 = c2.xy;        // the same result as "a"
```

So the vector functions let us use other vectors or parts of other vectors to build a new vector.

**Vector helpers:**

Speaking of "part of other vectors": Depending on what kind of data we use a vector for, we have different kinds of helpers to access this data.

For a position, each component of the vector means one coordinate for each axis, e.g. `vec3(3.0, 5.0, 9.0)` means `x=3.0, y=5.0, z = 9.0`. We can access them with the following helpers:

```glsl
vec3 position = vec3(3.0, 5.0, 9.0);

float x = position.x; // x gets the value 3.0
float y = position.y; // x gets the value 5.0
float z = position.z; // z gets the value 9.0

vec2 position2D = position.xy; // position2D gets the value vec2(3.0, 5.0)
vec2 flipped2D  = position.yx; // flipped2D gets the value vec2(5.0, 3.0), so x & y have been swapped

vec2 what = position.xz; // what gets the value vec2(3.0, 9.0)
```

So all combinations from `x`, `y` and `z` are possible if we want to mix them up.

We could change our vertex shader to make use of that:

```glsl
vec4 aVertexPos; // the input goes here: A single vertex (x,y, 0, 0) from the buffer
  
void main(void) {
  gl_Position = vec4(aVertexPos.yx, 1.0, 1.0);
}
```

As the result, we have swapped x- and y-coordinates:

![The triangle with swapped x- and y-coordinates](https://raw.githubusercontent.com/pluralsight/guides/master/images/dcb265e4-bdf6-40d7-afc4-a41e1ecfe57c.com_2016-08-05_10-27-48)

So now we know that we can use the vertex shader and the GLSL vector helpers to change the vertices we will be using for drawing - all without having to change data in our buffer.

Vertex shaders are usually used for things such as:

* scaling
* rotation
* perspective transformation (i.e. making things that are far away from the camera appear smaller and things that are closer to the camera appear larger)
* calculating texture coordinates (more on this later)

So we have been playing with the first of the two shaders, the vertex shader. But what about the fragment shader?

**The fragment shader:**

The fragment shader's job is to figure out what colour to put into the pixels inside our shape. It also uses a GLSL program to figure that out.

Right now our program is pretty boring: It always returns a simple colour for any pixel that is inside our fragment and will be drawn on screen:

```glsl
void main(void) {
  gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}
```

Here we see something, we already know: A four-component vector-
This time the values do not mean positions, but colour! So it turns out, that vectors are a pretty generic way to express data that has more than a single value, positions have coordinates on multiple axes, colours have values in different colour channels (red, green, blue and transparency).

We can totally use the helpers we've used for vectors that represent points like this:

```glsl
vec3 red   = vec3(1.0, 0.0, 0.0);
vec3 green = vec3(0.0, 1.0, 0.0);
vec3 blue  = vec3(0.0, 0.0, 1.0);

vec3 white = vec3(red.x, green.y, blue.z); // this looks weird!
```

That seems a little odd, because we are talking about colour, but use `x`, `y` and `z`...
There's good news, though: GLSL provides similar helpers for colours:

```glsl
vec3 white = vec3(red.r, green.g, blue.b); // this looks better!
```

There is also the possibility to swap components like we did with coordinates:
```glsl
vec3 red   = vec3(1.0, 0.0, 0.0);
vec3 green = red.grb; // returns vec3(0.0, 1.0, 0.0)
vec3 blue  = red.gbr; // returns vec3(0.0, 0.0, 1.0)
```

However, GLSL does not really *care* what data is in there, so you can always use any of these helpers - but it makes sense to use the way that fits the data, so use `r`, `g`, `b` and `a` (transparency is usually called "alpha") for colours and `x`, `y`, `z` for coordinates.

Now let's spice up the triangle a little by making it more colourful.
There is a global variable in fragment shaders called `gl_FragCoord` which [contains the four-component vector of the window coordinates](https://www.opengl.org/sdk/docs/man/html/gl_PointCoord.xhtml) of the current pixel that fragment shader is asked to compute, where (0, 0) is the bottom left of the canvas.

There is a bit of conversion needed there though: The `gl_FragCoord` has values between (0,0) and (499, 499) for our 500x500 pixel canvas, but our colours need to be values between 0.0 and 1.0, so we will divide the coordinates by the dimensions of the canvas to achieve useful values.

Now we can change our **fragment shader** to use x-coordinate of `gl_FragCoord` to determine how much red there should be:

```glsl
void main(void) {
  gl_FragColor = vec4(gl_FragCoord.x / 500.0, 0.0, 0.0, 1.0);
}
```

Here is the result:

![Using the fragment shader to produce a gradient](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd2a5701-a8f1-4447-98ba-702ada582e50.com_2016-08-05_11-04-33)

We can also use the y-coordinate of `gl_PointCoord` to do the same with green in vertical direction:

```glsl
void main(void) {
  gl_FragColor = vec4(0.0, gl_FragCoord.y / 500.0, 0.0, 1.0);
}
```

![Using the y-coordinate for the green value](https://raw.githubusercontent.com/pluralsight/guides/master/images/a5920d43-36ab-4ea4-942e-f145c9f3130c.com_2016-08-05_11-08-29)

And then mix the two:

```glsl
void main(void) {
  gl_FragColor = vec4(gl_FragCoord.y / 500.0, gl_FragCoord.y / 500.0, 0.0, 1.0);
}
```

![Using fragment coordinates for red and green](https://raw.githubusercontent.com/pluralsight/guides/master/images/2b24e54d-d311-49e9-9b96-7d45d0416cd2.com_2016-08-05_11-10-34)

# Going deep

So far we have only dealt with a single triangle so we were not facing the question: How do we handle depth?

The concept of depth allows us to figure out if an object (or part of it) is "in front" of another object (or part of it).

The way this is represented in WebGL is by using the z-coordinates and a bunch of memory referred to as the `depth buffer`.

Let's look at an example with two triangles:

A red one at `z=0.25` and a blue one at `z=0.5`. Here are the new vertices:

```javascript
var vertices = [
  // first triangle, red
  -0.75, -0.5, 0.25,
   0.25, -0.5, 0.25,
  -0.25,  0.5, 0.25,
  // second triangle, blue
  -0.25, -0.5, 0.5,
   0.75, -0.5, 0.5,
   0.25,  0.5, 0.5
]
```

Now we draw them using `drawArrays` again:

```javascript
gl.drawArrays(gl.TRIANGLES, 0, 6) // this time we draw 6 vertices
```

The question now is: Which of the triangles will be in front of the other?

Let's see:


![Two overlapping triangles, no depth buffer](https://raw.githubusercontent.com/pluralsight/guides/master/images/0978c373-6a9e-45d6-9956-4e367d58cd75.png)

The blue triangle, which was the last to be drawn, is drawn *over* the red triangle.
But this order is rather inflexible.

If the z-coordinates for the triangles would be flipped, it would look the same, because the colours are overriding previously set colours.

That's where the depth buffer comes in. The renderer saves the z-coordinate of the point along with the colour and only replaces it, if the new colour value is "closer" (more on that in a bit) to the camera.

Let's see how that works with three triangles with different z-coordinates:


![Using the depth buffer allows for more flexibility when drawing multiple objects](https://raw.githubusercontent.com/pluralsight/guides/master/images/1c6938a6-029c-4868-b086-af6f4694cea1.gif)

Now the order does not matter anymore, because the value in the depth buffer is determining what colour is being used in a given point. In the example we have the red triangle with `z=1` being drawn first, then the blue triangle with `z=1` paints over the red one and finally we draw a green triangle with `z=0` which is drawn as if it were behind the other two, even though it is the last to be drawn on the screen.

WebGL calls this option the `gl.DEPTH_TEST`. The depth test itself is basically a comparison between the depth buffer value that is already in the buffer. We have a bunch of options on how we want to do that comparison, called the *depth function*:

* `gl.ALWAYS` will always override the existing colour and depth value.
* `gl.NEVER` will never override the existing colour and depth value.
* `gl.LESS` will override the existing colour and depth value when the new z-coordinate is smaller than the existing depth value.
* `gl.GREATER` will override the existing colour and depth value when the new z-coordinate is greater than the existing depth value.
* `gl.EQUAL` will override the colour when the new z-coordinate is the same as the current depth value.
* `gl.NOTEQUAL` will override the colour and depth value when the new z-coordinate is not the same as the current depth value.
* `gl.LGREATER` will override the existing colour and depth value when the new z-coordinate is greater than or equal to the existing depth value.
* `gl.LEQUAL` will override the existing colour and depth value when the new z-coordinate is smaller than or equal to the existing depth value.

Basically it's up to us to pick our depth test function as we like and need. Assuming that larger values for the z-coordinate mean being closer to the camera, we will usually pick `gl.LEQUAL` or `gl.LESS`.

# Putting things into perspective

# Creating our first cube

# Rotating our cube
