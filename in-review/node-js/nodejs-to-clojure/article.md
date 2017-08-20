For those who have been living under the metaphorical rock, Clojure, a JVM-based Lisp dialect, is an up-and-coming server-side programming language that features immutable data structures and first-order functions, making it ideal for elegant and expressive functional patterns. ClojureScript is just Clojure that compiles to JavaScript; think CoffeeScript or TypeScript (Clojure has its own semantics separate from JavaScript, so not a one-to-one analogy here). ClojureScript (CLJS) has gained some traction in the React community, due to strong synergies between the two technologies and associated philosophies (check out [Reagent](https://github.com/reagent-project/reagent)). CLJS is primarily used for front-end development though, here, we will look at writing a simple Node.js server, using the JS/CLJS interop to interface NPM modules.

Lumo is a neat little Clojure-to-JS compiler that runs on Node.js. Though tools like Lieningen and Boot provide incredible valu  (like [REPL-based interactive programming](https://github.com/danielsz/holygrail)), they require some legwork to get running. Lumo really shines in this aspect: it’s dirt-cheap upfront, especially if you’re already running Node.js on your system.

With Lumo, getting a ClojureScript environment running is as simple as npm install -g lumo-cljs. Beautiful! We’re already getting to the good stuff. I’m actually gonna steal and modify [bit of code](https://gist.github.com/bhauman/c63123a5c655d77c3e7f) (thanks [bhauman](https://github.com/bhauman/), not all heroes wear capes!) to get a basic Express app serving GET /hello on port 3000. Note: You can pretty easily get Lumo to compile for the browser (making full use of Google's [Closure Compiler](https://developers.google.com/closure/compiler/) with optimizations like dead-code removal), and could even go as far as building an isomorphic React app on Lumo, but more on that in another post.

My server.cljs looks like this:
```clojure
(ns server.core
  (:require
   [cljs.nodejs :as nodejs]))

; Bring in `express` and `http` modules.
(defonce express (nodejs/require "express"))
(defonce http (nodejs/require "http"))

; Create our app.
(def app (express))

; Mount GET handler.
(. app (get "/hello"
  (fn [req res] (. res (send "Hello world")))))

; Listen on port 3000.
(doto (.createServer http #(app %1 %2))
  (.listen 3000))
```

[Looks familiar, huh?](https://expressjs.com/en/starter/hello-world.html) You’ll need Express installed to run the app, so go ahead and `npm install express`.
Ok, so next, you’ll need to… oh wait, I think that’s it?

```shell
$ lumo -i server.cljs
```

Now, in another terminal, `curl localhost:3000/hello`. Easy, right?
Lumo is incredibly fast to start up (unlike `lein repl`, for example), and in the JVM-dominated Clojure world, this is a very welcome change of pace. Automation scripts and CLIs in Clojure? Why not?
