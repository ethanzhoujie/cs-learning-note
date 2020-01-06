## The Evolution of Async JavaScript: From Callbacks, to Promises, to Async/Await

### Callback

This highlights some important concepts of the JavaScript language. First, just as you can pass a string or a number as an argument to a function, so too can you pass a reference to a function as an argument. When you do this the function you’re passing as an argument is called a **callback** function and the function you’re passing the callback function to is called a **higher order function**.

The first, and what we see in the `.map` and `_.filter`examples, is a nice abstraction over turning one value into another. 

The second, and what we see in the jQuery example, is delaying execution of a function until a particular time. 

### Promises

##### 1) How do you create a Promise?

This one is pretty straight forward. You create a `new` instance of `Promise`.

```js
const promise = new Promise()
```

##### 2) How do you change the status of a promise?

The `Promise` constructor function takes in a single argument, a (callback) function. This function is going to be passed two arguments, `resolve` and `reject`.

```
resolve` - a function that allows you to change the status of the promise to `fulfilled
```

`reject` - a function that allows you to change the status of the promise to `rejected`.

In the code below, we use `setTimeout` to wait 2 seconds and then invoke `resolve`. This will change the status of the promise to `fulfilled`.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve() // Change status to 'fulfilled'
  }, 2000)
})
```

We can see this change in action by logging the promise right after we create it and then again roughly 2 seconds later after `resolve` has been called.

![resolve](./assets/resolve.gif)

Notice the promise goes from `<pending>` to `<resolved>`.

#### 3) How do you listen for when the status of a promise changes?

In my opinion this is the most important question. It’s cool we know how to create a promise and change its status, but that’s worthless if we don’t know how to do anything after the status changes.

One thing we haven’t talked about yet is what a promise actually is. When you create a `new Promise`, you’re really just creating a plain old JavaScript object. This object can invoke two methods, `then`, and `catch`. Here’s the key. When the status of the promise changes to `fulfilled`, the function that was passed to `.then` will get invoked. When the status of a promise changes to `rejected`, the function that was passed to `.catch` will be invoked. What this means is that once you create a promise, you’ll pass the function you want to run if the async request is successful to `.then`. You’ll pass the function you want to run if the async request fails to `.catch`.

Let’s take a look at an example. We’ll use `setTimeout` again to change the status of the promise to `fulfilled`after two seconds (2000 milliseconds).

```js
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

```js
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

```js
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

#### Chaining

Both `.then` and `.catch` will return a new promise.

```js
function getUser(id) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: `https://api.github.com/users/${id}`,
      success: resolve,
      error: reject
    })
  })
}

function getWeather(user) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: getLocationURL(user.location.split(',')),
      success: resolve,
      error: reject,
    })
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis")
    .then(getWeather)
    .then((weather) => {
      // We need both the user and the weather here.
      // Right now we just have the weather
      updateUI() // ????
    })
    .catch(showError)
})
```

```js
function getWeather(user) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: getLocationURL(user.location.split(',')),
      success(weather) {
        resolve({ user, weather: weather.query.results })
      },
      error: reject,
    })
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis")
    .then(getWeather)
    .then((data) => {
      // Now, data is an object with a
      // "weather" property and a "user" property.

      updateUI(data)
    })
    .catch(showError)
})
```

It’s in our click handler where you really see the power of promises shine compared to callbacks.

```js
// Callbacks 🚫
getUser("tylermcginnis", (user) => {
  getWeather(user, (weather) => {
    updateUI({
      user,
      weather: weather.query.results
    })
  }, showError)
}, showError)


// Promises ✅
getUser("tylermcginnis")
  .then(getWeather)
  .then((data) => updateUI(data))
  .catch(showError);
```

