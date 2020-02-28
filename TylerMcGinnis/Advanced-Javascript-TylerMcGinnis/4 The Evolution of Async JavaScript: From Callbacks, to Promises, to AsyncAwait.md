## The Evolution of Async JavaScript: From Callbacks, to Promises, to Async/Await

### Callback

**Callback Pattern**

```js
function add (x, y) {
	return x + y;
}

function higherOrderFunction (x, callback) {
  return callback(x, 5);
}

higherOrderFunction(10, add);
```

This highlights some important concepts of the JavaScript language. 

First, just as you can pass a string or a number as an argument to a function, so too can you pass a reference to a function as an argument. When you do this the function you’re passing as an argument is called a **callback** function and the function you’re passing the callback function to is called a **higher order function**.

**Two use cases of callback:**

- The first, and what we see in the `.map` and `_.filter`examples, is a nice abstraction over turning one value into another. 


- The second, and what we see in the jQuery example, is delaying execution of a function until a particular time. 

**Callback Defects:**

- Callback Hell: Too many nested callbacks will cause readability issue.
- Inversion of Control: Give the control of callback function to the higher order function.

### Promises

##### 1) How do you create a Promise?

This one is pretty straight forward. You create a `new` instance of `Promise`.

```js
const promise = new Promise()
```

##### 2) How do you change the status of a promise?

The `Promise` constructor function takes in a single argument, a (callback) function. This function is going to be passed two arguments, `resolve` and `reject`.

`resolve` - a function that allows you to change the status of the promise to `fulfilled`

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

When you create a `new Promise`, you’re really just creating a plain old JavaScript object. This object can invoke two methods, `then`, and `catch`. 

- When the status of the promise changes to `fulfilled`, the function that was passed to `.then` will get invoked. 

- When the status of a promise changes to `rejected`, the function that was passed to `.catch` will be invoked. 

What this means is that once you create a promise, you’ll pass the function you want to run if the async request is successful to `.then`. You’ll pass the function you want to run if the async request fails to `.catch`.

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

#### async/await

Anytime you add `async` to a function, that function is going to implicitly return a promise.

```js
async function getPromise(){}
const promise = getPromise();
```

Even though `getPromise` is literally empty, it’ll still return a promise since it was an `async` function.

If the `async` function returns a value, that value will also get wrapped in a promise. That means you’ll have to use `.then` to access it.

```js
async function add (x, y) {
	return x + y;
}
add(2, 3).then((result) => {
  console.log(result) // 5
})
```

When add `async` to a function, it does two things:

- It makes it so the function itself returns a promise.

- It makes it so you can use `await` inside of it.

If use `await` without `async`, get an error

```js
$("#btn").on("click", () => {
  const user = await getUser('tylermcginnis') // SyntaxError: await is a reserved word
  const weather = await getWeather(user.location) // SyntaxError: await is a reserved word

  updateUI({
    user,
    weather,
  })
})
```

#### Error Handling

You may have noticed we cheated a little bit. In our original code we had a way to catch any errors using `.catch`. When we switched to Async/Await, we removed that code. 

With Async/Await, the most common approach is to wrap your code in a `try/catch` block to be able to catch the error.

```js
$("#btn").on("click", async () => {
  try {
    const user = await getUser('tylermcginnis')
    const weather = await getWeather(user.location)

    updateUI({
      user,
      weather,
    })
  } catch (e) {
    showError(e)
  }
})
```

