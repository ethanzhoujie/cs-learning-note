## From IIFEs to CommonJS to ES6 Modules

I’ve taught JavaScript for a long time to a lot of people. Consistently the most commonly under-learned aspect of the language is the module system. There’s a good reason for that. Modules in JavaScript have a strange and erratic history. In this post, we’ll walk through that history and you’ll learn modules of the past to better understand how JavaScript modules work today.

Before we learn how to create modules in JavaScript, we first need to understand what they are and why they exist. Look around you right now. Any marginally complex item that you can see is probably built using individual pieces that when put together, form the item.

Let’s take a watch for example.

![Internals of a watch](https://tylermcginnis.com/images/posts/advanced-javascript/watch-internals.jpg)

A simple wristwatch is made up of hundreds of internal pieces. Each has a specific purpose and clear boundaries for how it interacts with the other pieces. Put together, all of these pieces form the whole of the watch. Now I’m no watch engineer, but I think the benefits of this approach are pretty transparent.

#### Reusability

Take a look at the diagram above one more time. Notice how many of the same pieces are being used throughout the watch. Through highly intelligent design decisions centered on modularity, they’re able to re-use the same components throughout different aspects of the watch design. This ability to re-use pieces simplifies the manufacturing process and, I’m assuming, increases profit.

#### Composability

The diagram is a beautiful illustration of composability. By establishing clear boundaries for each individual component, they’re able to compose each piece together to create a fully functioning watch out of tiny, focused pieces.

#### Leverage

Think about the manufacturing process. This company isn’t making watches, it’s making individual components that together form a watch. They could create those pieces in house, they could outsource them and leverage other manufacturing plants, it doesn’t matter. The most important thing is that each piece comes together in the end to form a watch - where those pieces were created is irrelevant.

#### Isolation

Understanding the whole system is difficult. Because the watch is composed of small, focused pieces, each of those pieces can be thought about, built and or repaired in isolation. This isolation allows multiple people to work individually on the watch while not bottle-necking each other. Also if one of the pieces breaks, instead of replacing the whole watch, you just have to replace the individual piece that broke.

#### Organization

Organization is a byproduct of each individual piece having a clear boundary for how it interacts with other pieces. With this modularity, organization naturally occurs without much thought.

------

We’ve seen the obvious benefits of modularity when it comes to everyday items like a watch, but what about software? Turns out, it’s the same idea with the same benefits. Just how the watch was designed, we *should* design our software separated into different pieces where each piece has a specific purpose and clear boundaries for how it interacts with other pieces. In software, these pieces are called **modules**. At this point, a module might not sound too different from something like a function or a React component. So what exactly would a module encompass?

Each module has three parts - dependencies (also called imports), code, and exports.

```js
imports

code

exports
```

##### Dependencies (Imports)

When one module needs another module, it can `import` that module as a dependency. For example, whenever you want to create a React component, you need to `import` the `react` module. If you want to use a library like `lodash`, you’d need `import` the `lodash` module.

##### Code

After you’ve established what dependencies your module needs, the next part is the actual code of the module.

##### Exports

Exports are the “interface” to the module. Whatever you export from a module will be available to whoever imports that module.

------

Enough with the high-level stuff, let’s dive into some real examples.

First, let’s look at React Router. Conveniently, they have a [modules](https://github.com/ReactTraining/react-router/tree/master/packages/react-router/modules) folder. This folder is filled with… modules, naturally. So in React Router, what makes a “module”. Turns out, for the most part, they map their React components directly to modules. That makes sense and in general, is how you separate components in a React project. This works because if you re-read the watch above but swap out “module” with “component”, the metaphors still make sense.

Let’s look at the code from the `MemoryRouter` module. Don’t worry about the actual code for now, but focus on more of the structure of the module.

```js
// imports
import React from "react";
import { createMemoryHistory } from "history";
import Router from "./Router";

// code
class MemoryRouter extends React.Component {
  history = createMemoryHistory(this.props);
  render() {
    return (
      <Router
        history={this.history}
        children={this.props.children}
      />;
    )
  }
}

// exports
export default MemoryRouter;
```

You’ll notice at the top of the module they define their imports, or what other modules they need to make the `MemoryRouter` module work properly. Next, they have their code. In this case, they create a new React component called `MemoryRouter`. Then at the very bottom, they define their export, `MemoryRouter`. This means that whenever someone imports the `MemoryRouter` module, they’ll get the `MemoryRouter` component.

------

Now that we understand what a module is, let’s look back at the benefits of the watch design and see how, by following a similar modular architecture, those same benefits can apply to software design.

#### Reusability

Modules maximize reusability since a module can be imported and used in any other module that needs it. Beyond this, if a module would be beneficial in another program, you can create a package out of it. A package can contain one or more modules and can be uploaded to [NPM](https://npmjs.com/) to be downloaded by anyone. `react`, `lodash`, and `jquery` are all examples of NPM packages since they can be installed from the NPM directory.

#### Composability

Because modules explicitly define their imports and exports, they can be easily composed. More than that, a sign of good software is that is can be easily deleted. Modules increase the “delete-ability” of your code.

#### Leverage

The NPM registry hosts the world’s largest collection of free, reusable modules (over 700,000 to be exact). Odds are if you need a specific package, NPM has it.

#### Isolation

The text we used to describe the isolation of the watch fits perfectly here as well. “Understanding the whole system is difficult. Because (your software) is composed of small, focused (modules), each of those (modules) can be thought about, built and or repaired in isolation. This isolation allows multiple people to work individually on the (app) while not bottle-necking each other. Also if one of the (modules) breaks, instead of replacing the whole (app), you just have to replace the individual (module) that broke.”

#### Organization

Perhaps the biggest benefit in regards to modular software is organization. Modules provide a natural separation point. Along with that, as we’ll see soon, modules prevent you from polluting the global namespace and allow you to avoid naming collisions.

------

At this point, you know the benefits and understand the structure of modules. Now it’s time to actually start building them. Our approach to this will be pretty methodical. The reason for that is because, as mentioned earlier, modules in JavaScript have a strange history. Even though there are “newer” ways to create modules in JavaScript, some of the older flavors still exist and you’ll see them from time to time. If we jump straight to modules in 2018, I’d be doing you a disservice. With that said, we’re going to take it back to late 2010. AngularJS was just released and jQuery is all the rage. Companies are finally using JavaScript to build complex web applications and with that complexity comes a need to manage it - via modules.

Your first intuition for creating modules may be to separate code by files.

```js
// users.js
var users = ["Tyler", "Sarah", "Dan"]

function getUsers() {
  return users
}
// dom.js

function addUserToDOM(name) {
  const node = document.createElement("li")
  const text = document.createTextNode(name)
  node.appendChild(text)

  document.getElementById("users")
    .appendChild(node)
}

document.getElementById("submit")
  .addEventListener("click", function() {
    var input = document.getElementById("input")
    addUserToDOM(input.value)

    input.value = ""
})

var users = window.getUsers()
for (var i = 0; i < users.length; i++) {
  addUserToDOM(users[i])
}
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Users</title>
  </head>

  <body>
    <h1>Users</h1>
    <ul id="users"></ul>
    <input
      id="input"
      type="text"
      placeholder="New User">
    </input>
    <button id="submit">Submit</button>

    <script src="users.js"></script>
    <script src="dom.js"></script>
  </body>
</html>
```

> **The full code can be found [here](https://github.com/tylermcginnis/modules/tree/separate-files)**.

OK. We’ve successfully separated our app into its own files. Does that mean we’ve successfully implemented modules? No. Absolutely not. Literally, all we’ve done is separate where the code lives. The only way to create a new scope in JavaScript is with a function. All the variables we declared that aren’t in a function are just living on the global object. You can see this by logging the `window` object in the console. You’ll notice we can access, and worse, change `addUsers`, `users`, `getUsers`, `addUserToDOM`. That’s essentially our entire app. We’ve done nothing to separate our code into modules, all we’ve done is separate it by physical location. If you’re new to JavaScript, this may be a surprise to you but it was probably your first intuition for how to implement modules in JavaScript.

So if file separation doesn’t give us modules, what does? Remember the advantages to modules - reusability, composability, leverage, isolation, organization. Is there a native feature of JavaScript we could use to create our own “modules” that would give us the same benefits? What about a regular old function? When you think of the benefits of a function, they align nicely to the benefits of modules. So how would this work? What if instead of having our entire app live in the global namespace, we instead expose a single object, we’ll call it `APP`. We can then put all the methods our app needs to run under the `APP`, which will prevent us from polluting the global namespace. We could then wrap everything else in a function to keep it enclosed from the rest of the app.

```js
// App.js
var APP = {}
// users.js
function usersWrapper () {
  var users = ["Tyler", "Sarah", "Dan"]

  function getUsers() {
    return users
  }

  APP.getUsers = getUsers
}

usersWrapper()
// dom.js

function domWrapper() {
  function addUserToDOM(name) {
    const node = document.createElement("li")
    const text = document.createTextNode(name)
    node.appendChild(text)

    document.getElementById("users")
      .appendChild(node)
  }

  document.getElementById("submit")
    .addEventListener("click", function() {
      var input = document.getElementById("input")
      addUserToDOM(input.value)

      input.value = ""
  })

  var users = APP.getUsers()
  for (var i = 0; i < users.length; i++) {
    addUserToDOM(users[i])
  }
}

domWrapper()
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Users</title>
  </head>

  <body>
    <h1>Users</h1>
    <ul id="users"></ul>
    <input
      id="input"
      type="text"
      placeholder="New User">
    </input>
    <button id="submit">Submit</button>

    <script src="app.js"></script>    <script src="users.js"></script>
    <script src="dom.js"></script>
  </body>
</html>
```

> **The full code can be found [here](https://github.com/tylermcginnis/modules/tree/wrappers)**.

Now if you look at the `window` object, instead of it having all the important pieces of our app, it just has `APP` and our wrapper functions, `usersWrapper` and `domWrapper`. More important, none of our important code (like `users`) can be modified since they’re no longer on the global namespace.

Let’s see if we can take this a step further. Is there a way to get rid of our wrapper functions? Notice that we’re defining and then immediately invoking them. The only reason we gave them a name was so we could immediately invoke them. Is there a way to immediately invoke an anonymous function so we wouldn’t have to give them a name? Turns out there is and it even has a fancy name - `Immediately Invoked Function Expression` or `IIFE` for short.

### IIFE

Here’s what it looks like.

```js
(function () {
  console.log('Pronounced IF-EE')
})()
```

Notice it’s just an anonymous function expression that we’ve wrapped in parens ().

```js
(function () {
  console.log('Pronounced IF-EE')
})
```

Then, just like any other function, in order to invoke it, we add another pair of parens to the end of it.

```js
(function () {
  console.log('Pronounced IF-EE')
})()
```

Now let’s use our knowledge of IIFEs to get rid of our ugly wrapper functions and clean up the global namespace even more.

```js
// users.js

(function () {
  var users = ["Tyler", "Sarah", "Dan"]

  function getUsers() {
    return users
  }

  APP.getUsers = getUsers
})()
// dom.js

(function () {
  function addUserToDOM(name) {
    const node = document.createElement("li")
    const text = document.createTextNode(name)
    node.appendChild(text)

    document.getElementById("users")
      .appendChild(node)
  }

  document.getElementById("submit")
    .addEventListener("click", function() {
      var input = document.getElementById("input")
      addUserToDOM(input.value)

      input.value = ""
  })

  var users = APP.getUsers()
  for (var i = 0; i < users.length; i++) {
    addUserToDOM(users[i])
  }
})()
```

> **The full code can be found [here](https://github.com/tylermcginnis/modules/tree/IIFEs)**.

***chef’s kiss\***. Now if you look at the `window` object, you’ll notice the only thing we’ve added to it is `APP`, which we use as a namespace for all the methods our app needs to properly run.

Let’s call this pattern the **IIFE Module Pattern**.

What are the benefits to the IIFE Module Pattern? First and foremost, we avoid dumping everything onto the global namespace. This will help with variable collisions and keeps our code more private. Does it have any downsides? It sure does. We still have 1 item on the global namespace, `APP`. If by chance another library uses that same namespace, we’re in trouble. Second, you’ll notice the order of the `` tags in our `index.html` file matter. If you don’t have the scripts in the exact order they are now, the app will break.

Even though our solution isn’t perfect, we’re making progress. Now that we understand the pros and cons to the IIFE module pattern, if we were to make our own standard for creating and managing modules, what features would it have?

Earlier our first instinct for the separation of modules was to have a new module for each file. Even though that doesn’t work out of the box with JavaScript, I think that’s an obvious separation point for our modules. **Each file is its own module.** Then from there, the only other feature we’d need is to have each file define **explicit imports** (or dependencies) and **explicit exports** which will be available to any other file that imports the module.

```js
Our Module Standard

1) File based
2) Explicit imports
3) Explicit exports
```

Now that we know the features our module standard will need, let’s dive into the API. The only real API we need to define is what imports and exports look like. Let’s start with exports. To keep things simple, any information regarding the module can go on the `module` object. Then, anything we want to export from a module we can stick on `module.exports`. Something like this

```js
var users = ["Tyler", "Sarah", "Dan"]

function getUsers() {
  return users
}

module.exports.getUsers = getUsers
```

This means another way we can write it is like this

```js
var users = ["Tyler", "Sarah", "Dan"]

function getUsers() {
  return users
}

module.exports = {
  getUsers: getUsers
}
```

Regardless of how many methods we had, we could just add them to the `exports` object.

```js
// users.js

var users = ["Tyler", "Sarah", "Dan"]

module.exports = {
  getUsers: function () {
    return users
  },
  sortUsers: function () {
    return users.sort()
  },
  firstUser: function () {
    return users[0]
  }
}
```

Now that we’ve figured out what exporting from a module looks like, we need to figure out what the API for importing modules looks like. To keep this one simple as well, let’s pretend we had a function called `require`. It’ll take a string path as its first argument and will return whatever is being exported from that path. Going along with our `users.js` file above, to import that module would look something like this

```js
var users = require('./users')

users.getUsers() // ["Tyler", "Sarah", "Dan"]
users.sortUsers() // ["Dan", "Sarah", "Tyler"]
users.firstUser() // ["Tyler"]
```

Pretty slick. With our hypothetical `module.exports` and `require` syntax, we’ve kept all of the benefits of modules while getting rid of the two downsides to our IIFE Modules pattern.

As you probably guessed by now, this isn’t a made up standard. It’s real and it’s called CommonJS.

> The CommonJS group defined a module format to solve JavaScript scope issues by making sure each module is executed in its own namespace. This is achieved by forcing modules to explicitly export those variables it wants to expose to the “universe”, and also by defining those other modules required to properly work.
>
> \- Webpack docs

If you’ve used Node before, CommonJS should look familiar. The reason for that is because Node uses (for the most part) the CommonJS specification in order to implement modules. So with Node, you get modules out of the box using the CommonJS `require` and `module.exports` syntax you saw earlier. However, unlike Node, browsers don’t support CommonJS. In fact, not only do browsers not support CommonJS, but out of the box, CommonJS isn’t a great solution for browsers since it loads modules synchronously. In the land of the browser, the asynchronous loader is king.

So in summary, there are two problems with CommonJS. First, the browser doesn’t understand it. Second, it loads modules synchronously which in the browser would be a terrible user experience. If we can fix those two problems, we’re in good shape. So what’s the point of spending all this time talking about CommonJS if it’s not even good for browsers? Well, there is a solution and it’s called a module bundler.

### Module Bundlers

What a JavaScript module bundler does is it examines your codebase, looks at all the imports and exports, then intelligently bundles all of your modules together into a single file that the browser can understand. Then instead of including all the scripts in your index.html file and worrying about what order they go in, you include the single `bundle.js` file the bundler creates for you.

```js
app.js ---> |         |
users.js -> | Bundler | -> bundle.js
dom.js ---> |         |
```

So how does a bundler actually work? That’s a really big question and one I don’t fully understand myself, but here’s the output after running our simple code through [Webpack](https://webpack.js.org/), a popular module bundler.

> **The full code can with CommonJS and Webpack can be found [here](https://github.com/tylermcginnis/modules/tree/commonjs)**. You’ll need to download the code, run “npm install”, then run “webpack”.

```js
(function(modules) { // webpackBootstrap
  // The module cache
  var installedModules = {};
  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if(installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    };
    // Execute the module function
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    );
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
  }
  // expose the modules object (__webpack_modules__)
  __webpack_require__.m = modules;
  // expose the module cache
  __webpack_require__.c = installedModules;
  // define getter function for harmony exports
  __webpack_require__.d = function(exports, name, getter) {
    if(!__webpack_require__.o(exports, name)) {
      Object.defineProperty(
        exports,
        name,
        { enumerable: true, get: getter }
      );
    }
  };
  // define __esModule on exports
  __webpack_require__.r = function(exports) {
    if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };
  // create a fake namespace object
  // mode & 1: value is a module id, require it
  // mode & 2: merge all properties of value into the ns
  // mode & 4: return value when already ns object
  // mode & 8|1: behave like require
  __webpack_require__.t = function(value, mode) {
    if(mode & 1) value = __webpack_require__(value);
    if(mode & 8) return value;
    if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
    var ns = Object.create(null);
    __webpack_require__.r(ns);
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if(mode & 2 && typeof value != 'string')
      for(var key in value)
        __webpack_require__.d(ns, key, function(key) {
          return value[key];
        }.bind(null, key));
    return ns;
  };
  // getDefaultExport function for compatibility with non-harmony modules
  __webpack_require__.n = function(module) {
    var getter = module && module.__esModule ?
      function getDefault() { return module['default']; } :
      function getModuleExports() { return module; };
    __webpack_require__.d(getter, 'a', getter);
    return getter;
  };
  // Object.prototype.hasOwnProperty.call
  __webpack_require__.o = function(object, property) {
      return Object.prototype.hasOwnProperty.call(object, property);
  };
  // __webpack_public_path__
  __webpack_require__.p = "";
  // Load entry module and return exports
  return __webpack_require__(__webpack_require__.s = "./dom.js");
})
/************************************************************************/
({

/***/ "./dom.js":
/*!****************!*\
  !*** ./dom.js ***!
  \****************/
/*! no static exports found */
/***/ (function(module, exports, __webpack_require__) {

eval(`
  var getUsers = __webpack_require__(/*! ./users */ \"./users.js\").getUsers\n\n
  function addUserToDOM(name) {\n
    const node = document.createElement(\"li\")\n
    const text = document.createTextNode(name)\n
    node.appendChild(text)\n\n
    document.getElementById(\"users\")\n
      .appendChild(node)\n}\n\n
    document.getElementById(\"submit\")\n
      .addEventListener(\"click\", function() {\n
        var input = document.getElementById(\"input\")\n
        addUserToDOM(input.value)\n\n
        input.value = \"\"\n})\n\n
        var users = getUsers()\n
        for (var i = 0; i < users.length; i++) {\n
          addUserToDOM(users[i])\n
        }\n\n\n//# sourceURL=webpack:///./dom.js?`
);}),

/***/ "./users.js":
/*!******************!*\
  !*** ./users.js ***!
  \******************/
/*! no static exports found */
/***/ (function(module, exports) {

eval(`
  var users = [\"Tyler\", \"Sarah\", \"Dan\"]\n\n
  function getUsers() {\n
    return users\n}\n\nmodule.exports = {\n
      getUsers: getUsers\n
    }\n\n//# sourceURL=webpack:///./users.js?`);})
});
```

You’ll notice that there’s a lot of magic going on there (you can read the comments if you want to know exactly what’s happening), but one thing that’s interesting is they wrap all the code inside of a big IIFE. So they’ve figured out a way to get all of the benefits of a nice module system without the downsides, simply by utilizing our old IIFE Module Pattern.

------

What really future proofs JavaScript is that it’s a living language. TC-39, the standards committee around JavaScript, meets a few times a year to discuss potential improvements to the language. At this point, it should be pretty clear that modules are a critical feature for writing scalable, maintainable JavaScript. In ~2013 (and probably long before) it was dead obvious that JavaScript needed a standardized, built in solution for handling modules. This kicked off the process for implementing modules natively into JavaScript.

Knowing what you know now, if you were tasked with creating a module system for JavaScript, what would it look like? CommonJS got it mostly right. Like CommonJS, each file could be a new module with a clear way to define imports and exports - obviously, that’s the whole point. A problem we ran into with CommonJS is it loads modules synchronously. That’s great for the server but not for the browser. One change we could make would be to support asynchronous loading. Another change we could make is rather than a `require` function call, since we’re talking about adding to the language itself, we could define new keywords. Let’s go with `import` and `export`.

Without going too far down the “hypothetical, made up standard” road again, the TC-39 committee came up with these exact same design decisions when they created “ES Modules”, now the standardized way to create modules in JavaScript. Let’s take a look at the syntax.

### ES Modules

As mentioned above, to specify what should be exported from a module you use the `export` keyword.

```js
// utils.js

// Not exported
function once(fn, context) {
  var result
  return function() {
    if(fn) {
      result = fn.apply(context || this, arguments)
      fn = null
    }
    return result
  }
}

// Exported
export function first (arr) {
  return arr[0]
}

// Exported
export function last (arr) {
  return arr[arr.length - 1]
}
```

Now to import `first` and `last`, you have a few different options. One is to import everything that is being exported from `utils.js`.

```js
import * as utils from './utils'

utils.first([1,2,3]) // 1
utils.last([1,2,3]) // 3
```

But what if we didn’t want to import everything the module is exporting? In this example, what if we wanted to import `first` but not `last`? This is where you can use what’s called `named imports` (it looks like destructuring but it’s not).

```js
import { first } from './utils'

first([1,2,3]) // 1
```

What’s cool about ES Modules is not only can you specify multiple exports, but you can also specify a `default` export.

```js
// leftpad.js

export default function leftpad (str, len, ch) {
  var pad = '';
  while (true) {
    if (len & 1) pad += ch;
    len >>= 1;
    else break;
  }
  return pad + str;
}
```

When you use a `default` export, that changes how you import that module. Instead of using the `*` syntax or using named imports, you just use `import name from './path'`.

```js
import leftpad from './leftpad'
```

Now, what if you had a module that was exporting a `default` export but also other regular exports as well? Well, you’d do it how you’d expect.

```js
// utils.js

function once(fn, context) {
  var result
  return function() {
    if(fn) {
      result = fn.apply(context || this, arguments)
      fn = null
    }
    return result
  }
}

// regular export
export function first (arr) {
  return arr[0]
}

// regular export
export function last (arr) {
  return arr[arr.length - 1]
}

// default export
export default function leftpad (str, len, ch) {
  var pad = '';
  while (true) {
    if (len & 1) pad += ch;
    len >>= 1;
    else break;
  }
  return pad + str;
}
```

Now, what would the import syntax look like? In this case, again, it should be what you expect.

```js
import leftpad, { first, last } from './utils'
```

Pretty slick, yeah? `leftpad` is the `default` export and `first` and `last` are just the regular exports.

What’s interesting about ES Modules is, because they’re now native to JavaScript, modern browsers support them without using a bundler. Let’s look back at our simple Users example from the beginning of this tutorial and see what it would look like with ES Modules.

> **The full code can be found [here](https://github.com/tylermcginnis/modules/tree/esModules)**.

```js
// users.js

var users = ["Tyler", "Sarah", "Dan"]

export default function getUsers() {
  return users
}
// dom.js

import getUsers from './users.js'

function addUserToDOM(name) {
  const node = document.createElement("li")
  const text = document.createTextNode(name)
  node.appendChild(text)

  document.getElementById("users")
    .appendChild(node)
}

document.getElementById("submit")
  .addEventListener("click", function() {
    var input = document.getElementById("input")
    addUserToDOM(input.value)

    input.value = ""
})

var users = getUsers()
for (var i = 0; i < users.length; i++) {
  addUserToDOM(users[i])
}
```

Now here’s the cool part. With our IIFE pattern, we still needed to include a script to every JS file (and in order, none the less). With CommonJS we needed to use a bundler like Webpack and then include a script to the `bundle.js` file. With ES Modules, in [modern browsers](https://caniuse.com/#feat=es6-modulef), all we need to do is include our main file (in this case `dom.js`) and add a `type='module'` attribute to the script tab.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Users</title>
  </head>

  <body>
    <h1>Users</h1>
    <ul id="users">
    </ul>
    <input id="input" type="text" placeholder="New User"></input>
    <button id="submit">Submit</button>

    <script type=module src='dom.js'></script>  </body>
</html>
```

------

### Tree Shaking

There’s one more difference between CommonJS modules and ES Modules that we didn’t cover above.

With CommonJS, you can `require` a module anywhere, even conditionally.

```js
if (pastTheFold === true) {
  require('./parallax')
}
```

Because ES Modules are static, import statements must always be at the top level of a module. You can’t conditionally import them.

```js
if (pastTheFold === true) {
  import './parallax' // "import' and 'export' may only appear at the top level"
}
```

The reason this design decision was made was because by forcing modules to be static, the loader can statically analyze the module tree, figure out which code is actually being used, and drop the unused code from your bundle. That was a lot of big words. Said differently, because ES Modules force you to declare your import statements at the top of your module, the bundler can quickly understand your dependency tree. When it understands your dependency tree, it can see what code isn’t being used and drop it from the bundle. This is called [Tree Shaking or Dead Code Elimination](https://webpack.js.org/guides/tree-shaking/).

> There is a stage 3 proposal for [dynamic imports](https://tylermcginnis.com/react-router-code-splitting/) which will allow you to conditionally load modules via import().

------

I hope diving into the history of JavaScript modules has helped you gain not only a better appreciation for ES Modules, but also a better understanding of their design decisions.