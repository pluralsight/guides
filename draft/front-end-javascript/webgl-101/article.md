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
* The two scripts with type `text/plain` are not executed by the browser. We will use them to hold the source code of the so-called "shaders" we need for rendering later.

# Creating our first triangle

# Writing our first shaders

# Creating our first cube

# Rotating our cube

# Putting things into perspective


