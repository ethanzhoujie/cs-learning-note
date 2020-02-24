## Callbacks, Promises, and Async/Await

One of my favorite sites is [BerkshireHathaway.com](http://www.berkshirehathaway.com/) - it’s simple, effective, and has been doing its job well since it launched in 1997. Even more remarkable, over the last 20 years, there’s a good chance this site has never had a bug. Why? Because it’s all static. It’s been pretty much the same since it launched over 20 years ago. Turns out sites are pretty simple to build if you have all of your data up front. Unfortunately, most sites now days don’t. To compensate for this, we’ve invented “patterns” for handling fetching external data for our apps. Like most things, these patterns each have tradeoffs that have changed over time. In this post, we’ll break down the pros and cons of three of the most common patterns, `Callbacks`, `Promises`, and `Async/Await` and talk about their significance and progression from a historical context.

Let’s start with the OG of these data fetching patterns, Callbacks.

### Callbacks

> I’m going to assume you know exactly 0 about callbacks. If I’m assuming wrong, just scroll down a bit.

When I was first learning to program, it helped me to think about functions as machines. These machines can do anything you want them to. They can even accept input and return a value. Each machine has a button on it that you can press when you want the machine to run, ().

```js
function add (x, y) {
  return x + y
}

add(2,3) // 5 - Press the button, run the machine.
```

Whether **I** press the button, **you** press the button, or **someone else** presses the button doesn’t matter. Whenever the button is pressed, like it or not, the machine is going to run.

```js
function add (x, y) {
  return x + y
}

const me = add
const you = add
const someoneElse = add

me(2,3) // 5 - Press the button, run the machine.
you(2,3) // 5 - Press the button, run the machine.
someoneElse(2,3) // 5 - Press the button, run the machine.
```

In the code above we assign the `add` function to three different variables, `me`, `you`, and `someoneElse`. It’s important to note that the original `add` and each of the variables we created are pointing to the same spot in memory. They’re literally the exact same thing under different names. So when we invoke `me`, `you`, or `someoneElse`, it’s as if we’re invoking `add`.

Now, what if we take our `add` machine and pass it to another machine? Remember, it doesn’t matter who presses the () button, if it’s pressed, it’s going to run.

```js
function add (x, y) {
  return x + y
}

function addFive (x, addReference) {
  return addReference(x, 5) // 15 - Press the button, run the machine.
}

addFive(10, add) // 15
```

Your brain might have got a little weird on this one, nothing new is going on here though. Instead of “pressing the button” on `add`, we pass `add` as an argument to `addFive`, rename it `addReference`, and then we “press the button” or invoke it.

This highlights some important concepts of the JavaScript language. First, just as you can pass a string or a number as an argument to a function, so too can you pass a reference to a function as an argument. When you do this the function you’re passing as an argument is called a **callback** function and the function you’re passing the callback function to is called a **higher order function**.

Because vocabulary is important, here’s the same code with the variables re-named to match the concepts they’re demonstrating.

```js
function add (x,y) {
  return x + y
}

function higherOrderFunction (x, callback) {
  return callback(x, 5)
}

higherOrderFunction(10, add)
```

This pattern should look familiar, it’s everywhere. If you’ve ever used any of the JavaScript [Array methods](https://tylermcginnis.com/javascript-array-methods-you-should-know/), you’ve used a callback. If you’ve ever used lodash, you’ve used a callback. If you’ve ever used jQuery, you’ve used a callback.

```js
[1,2,3].map((i) => i + 5)

_.filter([1,2,3,4], (n) => n % 2 === 0 );

$('#btn').on('click', () =>
  console.log('Callbacks are everywhere')
)
```

In general, there are two popular use cases for callbacks. The first, and what we see in the `.map` and `_.filter` examples, is a nice abstraction over transforming one value into another. We say “Hey, here’s an array and a function. Go ahead and get me a new value based on the function I gave you”. The second, and what we see in the jQuery example, is delaying execution of a function until a particular time. “Hey, here’s this function. Go ahead and invoke it whenever the element with an id of `btn` is clicked.” It’s this second use case that we’re going to focus on, “delaying execution of a function until a particular time”.

Right now we’ve only looked at examples that are synchronous. As we talked about at the beginning of this post, most of the apps we build don’t have all the data they need up front. Instead, they need to fetch external data as the user interacts with the app. We’ve just seen how callbacks can be a great use case for this because, again, they allow you to “delay execution of a function until a particular time”. It doesn’t take much imagination to see how we can adapt that sentence to work with data fetching. Instead of delaying execution of a function until *a particular time*, we can delay execution of a function *until we have the data we need*. Here’s probably the most popular example of this, jQuery’s `getJSON` method.

```js
// updateUI and showError are irrelevant.
// Pretend they do what they sound like.

const id = 'tylermcginnis'

$.getJSON({
  url: `https://api.github.com/users/${id}`,
  success: updateUI,
  error: showError,
})
```

We can’t update the UI of our app until we have the user’s data. So what do we do? We say, “Hey, here’s an object. If the request succeeds, go ahead and call `success` passing it the user’s data. If it doesn’t, go ahead and call `error` passing it the error object. You don’t need to worry about what each method does, just be sure to call them when you’re supposed to”. This is a perfect demonstration of using a callback for async requests.

------

At this point, we’ve learned about what callbacks are and how they can be beneficial both in synchronous and asynchronous code. What we haven’t talked yet is the dark side of callbacks. Take a look at this code below. Can you tell what’s happening?

```js
// updateUI, showError, and getLocationURL are irrelevant.
// Pretend they do what they sound like.

const id = 'tylermcginnis'

$("#btn").on("click", () => {
  $.getJSON({
    url: `https://api.github.com/users/${id}`,
    success: (user) => {
      $.getJSON({
        url: getLocationURL(user.location.split(',')),
        success (weather) {
          updateUI({
            user,
            weather: weather.query.results
          })
        },
        error: showError,
      })
    },
    error: showError
  })
})
```

If it helps, you can play around with the [live version here](https://codesandbox.io/s/v06mmo3j7l).

Notice we’ve added a few more layers of callbacks. First, we’re saying don’t run the initial AJAX request until the element with an id of `btn` is clicked. Once the button is clicked, we make the first request. If that request succeeds, we make a second request. If that request succeeds, we invoke the `updateUI` method passing it the data we got from both requests. Regardless of if you understood the code at first glance or not, objectively it’s much harder to read than the code before. This brings us to the topic of “Callback Hell”.

As humans, we naturally think sequentially. When you have nested callbacks inside of nested callbacks, it forces you out of your natural way of thinking. Bugs happen when there’s a disconnect between how your software is read and how you naturally think.

Like most solutions to software problems, a commonly prescribed approach for making “Callback Hell” easier to consume is to modularize your code.

```js
function getUser(id, onSuccess, onFailure) {
  $.getJSON({
    url: `https://api.github.com/users/${id}`,
    success: onSuccess,
    error: onFailure
  })
}

function getWeather(user, onSuccess, onFailure) {
  $.getJSON({
    url: getLocationURL(user.location.split(',')),
    success: onSuccess,
    error: onFailure,
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis", (user) => {
    getWeather(user, (weather) => {
      updateUI({
        user,
        weather: weather.query.results
      })
    }, showError)
  }, showError)
})
```

If it helps, you can play around with the [live version here](https://codesandbox.io/s/m587rq0lox).

OK, the function names help us understand what’s going on, but is it objectively “better”? Not by much. We’ve put a band-aid over the readability issue of Callback Hell. The problem still exists that we naturally think sequentially and, even with the extra functions, nested callbacks break us out of that sequential way of thinking.

------

The next issue of callbacks has to do with [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control). When you write a callback, you’re assuming that the program you’re giving the callback to is responsible and will call it when (and only when) it’s supposed to. You’re essentially inverting the control of your program over to another program. When you’re dealing with libraries like jQuery, lodash, or even vanilla JavaScript, it’s safe to assume that the callback function will be invoked at the correct time with the correct arguments. However, for many third-party libraries, callback functions are the interface for how you interact with them. It’s entirely plausible that a third party library could, whether on purpose or accidentally, break how they interact with your callback.

```js
function criticalFunction () {
  // It's critical that this function
  // gets called and with the correct
  // arguments.
}

thirdPartyLib(criticalFunction)
```

Since you’re not the one calling `criticalFunction`, you have 0 control over when and with what argument it’s invoked. *Most* of the time this isn’t an issue, but when it is, it’s a big one.

------

### Promises

Have you ever been to a busy restaurant without a reservation? When this happens, the restaurant needs a way to get back in contact with you when a table opens up. Historically, they’d just take your name and yell it when your table was ready. Then, as naturally occurs, they decided to start getting fancy. One solution was, instead of taking your name, they’d take your number and text you once a table opened up. This allowed you to be out of yelling range but more importantly, it allowed them to target your phone with ads whenever they wanted. Sound familiar? It should! OK, maybe it shouldn’t. It’s a metaphor for callbacks! **Giving your number to a restaurant is just like giving a callback function to a third party service. You \*expect\* the restaurant to text you when a table opens up, just like you \*expect\* the third party service to invoke your function when and how they said they would.** Once your number or callback function is in their hands though, you’ve lost all control.

Thankfully, there is another solution that exists. One that, by design, allows you to keep all the control. You’ve probably even experienced it before - it’s that little buzzer thing they give you. You know, this one.

![Restaurant Buzzer](https://tylermcginnis.com/images/posts/advanced-javascript/buzzy.png)

If you’ve never used one before, the idea is simple. Instead of taking your name or number, they give you this device. When the device starts buzzing and glowing, your table is ready. You can still do whatever you’d like as you’re waiting for your table to open up, but now you don’t have to give up anything. In fact, it’s the exact opposite. **They** have to give **you** something. There is no inversion of control.

The buzzer will always be in one of three different states - `pending`, `fulfilled`, or `rejected`.

`pending` is the default, initial state. When they give you the buzzer, it’s in this state.

`fulfilled` is the state the buzzer is in when it’s flashing and your table is ready.

`rejected` is the state the buzzer is in when something goes wrong. Maybe the restaurant is about to close or they forgot someone rented out the restaurant for the night.

Again, the important thing to remember is that you, the receiver of the buzzer, have all the control. If the buzzer gets put into `fulfilled`, you can go to your table. If it gets put into `fulfilled` and you want to ignore it, cool, you can do that too. If it gets put into `rejected`, that sucks but you can go somewhere else to eat. If nothing ever happens and it stays in `pending`, you never get to eat but you’re not actually out anything.

Now that you’re a master of the restaurant buzzer thingy, let’s apply that knowledge to something that matters.

**If giving the restaurant your number is like giving them a callback function, receiving the little buzzy thing is like receiving what’s called a “Promise”.**

As always, let’s start with **why**. Why do Promises exist? They exist to make the complexity of making asynchronous requests more manageable. Exactly like the buzzer, a `Promise` can be in one of three states, `pending`, `fulfilled` or `rejected`. Unlike the buzzer, instead of these states representing the status of a table at a restaurant, they represent the status of an asynchronous request.

If the async request is still ongoing, the `Promise` will have a status of `pending`. If the async request was successfully completed, the `Promise` will change to a status of `fulfilled`. If the async request failed, the `Promise` will change to a status of `rejected`. The buzzer metaphor is pretty spot on, right?

Now that you understand why Promises exist and the different states they can be in, there are three more questions we need to answer.

1. How do you create a Promise?
2. How do you change the status of a promise?
3. How do you listen for when the status of a promise changes?

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

![Use resolve to change the status of a promise to fulfilled](https://tylermcginnis.com/images/posts/advanced-javascript/resolve.gif)

Notice the promise goes from `` to ``.

#### 3) How do you listen for when the status of a promise changes?

In my opinion, this is the most important question. It’s cool we know how to create a promise and change its status, but that’s worthless if we don’t know how to do anything after the status changes.

One thing we haven’t talked about yet is what a promise actually is. When you create a `new Promise`, you’re really just creating a plain old JavaScript object. This object can invoke two methods, `then`, and `catch`. Here’s the key. When the status of the promise changes to `fulfilled`, the function that was passed to `.then` will get invoked. When the status of a promise changes to `rejected`, the function that was passed to `.catch` will be invoked. What this means is that once you create a promise, you’ll pass the function you want to run if the async request is successful to `.then`. You’ll pass the function you want to run if the async request fails to `.catch`.

Let’s take a look at an example. We’ll use `setTimeout` again to change the status of the promise to `fulfilled` after two seconds (2000 milliseconds).

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

If you run the code above you’ll notice that roughly 2 seconds later, you’ll see “Success!” in the console. Again the reason this happens is because of two things. First, when we created the promise, we invoked `resolve` after ~2000 milliseconds - this changed the status of the promise to `fulfilled`. Second, we passed the `onSuccess` function to the promises’ `.then` method. By doing that we told the promise to invoke `onSuccess` when the status of the promise changed to `fulfilled` which it did after ~2000 milliseconds.

Now let’s pretend something bad happened and we wanted to change the status of the promise to `rejected`. Instead of calling `resolve`, we would call `reject`.

```js
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

Now this time instead of the `onSuccess` function being invoked, the `onError` function will be invoked since we called `reject`.

------

Now that you know your way around the Promise API, let’s start looking at some real code.

Remember the last async callback example we saw earlier?

```js
function getUser(id, onSuccess, onFailure) {
  $.getJSON({
    url: `https://api.github.com/users/${id}`,
    success: onSuccess,
    error: onFailure
  })
}

function getWeather(user, onSuccess, onFailure) {
  $.getJSON({
    url: getLocationURL(user.location.split(',')),
    success: onSuccess,
    error: onFailure,
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis", (user) => {
    getWeather(user, (weather) => {
      updateUI({
        user,
        weather: weather.query.results
      })
    }, showError)
  }, showError)
})
```

Is there any way we could use the Promise API here instead of using callbacks? What if we wrap our AJAX requests inside of a promise? Then we can simply `resolve` or `reject` depending on how the request goes. Let’s start with `getUser`.

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
```

Nice. Notice that the parameters of `getUser` have changed. Instead of receiving `id`, `onSuccess`, and `onFailure`, it just receives `id`. There’s no more need for those other two callback functions because we’re no longer inverting control. Instead, we use the Promise’s `resolve` and `reject` functions. `resolve` will be invoked if the request was successful, `reject` will be invoked if there was an error.

Next, let’s refactor `getWeather`. We’ll follow the same strategy here. Instead of taking in `onSuccess` and `onFailure` callback functions, we’ll use `resolve` and `reject`.

```js
function getWeather(user) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: getLocationURL(user.location.split(',')),
      success: resolve,
      error: reject,
    })
  })
}
```

Looking good. Now the last thing we need to update is our click handler. Remember, here’s the flow we want to take.

1. Get the user’s information from the Github API.
2. Use the user’s location to get their weather from the Yahoo Weather API.
3. Update the UI with the user’s info and their weather.

Let’s start with #1 - getting the user’s information from the Github API.

```js
$("#btn").on("click", () => {
  const userPromise = getUser('tylermcginnis')

  userPromise.then((user) => {

  })

  userPromise.catch(showError)
})
```

Notice that now instead of `getUser` taking in two callback functions, it returns us a promise that we can call `.then` and `.catch` on. If `.then` is called, it’ll be called with the user’s information. If `.catch` is called, it’ll be called with the error.

Next, let’s do #2 - Use the user’s location to get their weather.

```js
$("#btn").on("click", () => {
  const userPromise = getUser('tylermcginnis')

  userPromise.then((user) => {
    const weatherPromise = getWeather(user)
    weatherPromise.then((weather) => {

    })

    weatherPromise.catch(showError)
  })

  userPromise.catch(showError)
})
```

Notice we follow the exact same pattern we did in #1 but now we invoke `getWeather` passing it the `user` object we got from `userPromise`.

Finally, #3 - Update the UI with the user’s info and their weather.

```js
$("#btn").on("click", () => {
  const userPromise = getUser('tylermcginnis')

  userPromise.then((user) => {
    const weatherPromise = getWeather(user)
    weatherPromise.then((weather) => {
      updateUI({
        user,
        weather: weather.query.results
      })
    })

    weatherPromise.catch(showError)
  })

  userPromise.catch(showError)
})
```

[Here’s the full code](https://codesandbox.io/s/l9xrjq88p7) you can play around with.

Our new code is *better*, but there are still some improvements we can make. Before we can make those improvements though, there are two more features of promises you need to be aware of, chaining and passing arguments from `resolve` to `then`.

#### Chaining

Both `.then` and `.catch` will return a new promise. That seems like a small detail but it’s important because it means that promises can be chained.

In the example below, we call `getPromise` which returns us a promise that will resolve in at least 2000 milliseconds. From there, because `.then` will return a promise, we can continue to chain our `.then`s together until we throw a `new Error` which is caught by the `.catch` method.

```js
function getPromise () {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000)
  })
}

function logA () {
  console.log('A')
}

function logB () {
  console.log('B')
}

function logCAndThrow () {
  console.log('C')

  throw new Error()
}

function catchError () {
  console.log('Error!')
}

getPromise()
  .then(logA) // A
  .then(logB) // B
  .then(logCAndThrow) // C
  .catch(catchError) // Error!
```

Cool, but why is this so important? Remember back in the callback section we talked about one of the downfalls of callbacks being that they force you out of your natural, sequential way of thinking. When you chain promises together, it doesn’t force you out of that natural way of thinking because chained promises are sequential. `getPromise runs then logA runs then logB runs then...`.

Just so you can see one more example, here’s a common use case when you use the `fetch` API. `fetch` will return you a promise that will resolve with the HTTP response. To get the actual JSON, you’ll need to call `.json`. Because of chaining, we can think about this in a sequential manner.

```js
fetch('/api/user.json')
  .then((response) => response.json())
  .then((user) => {
    // user is now ready to go.
  })
```

Now that we know about chaining, let’s refactor our `getUser`/`getWeather` code from earlier to use it.

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

It *looks* much better, but now we’re running into an issue. Can you spot it? In the second `.then` we want to call `updateUI`. The problem is we need to pass `updateUI` both the `user` and the `weather`. Currently, how we have it set up, we’re only receiving the `weather`, not the `user`. Somehow we need to figure out a way to make it so the promise that `getWeather` returns is resolved with both the `user` and the `weather`.

Here’s the key. `resolve` is just a function. Any arguments you pass to it will be passed along to the function given to `.then`. What that means is that inside of `getWeather`, if we invoke `resolve` ourself, we can pass to it `weather` and `user`. Then, the second `.then` method in our chain will receive both `user` and `weather` as an argument.

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

> You can play around with the [final code here](https://codesandbox.io/s/9lkl75vqxw)

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

Following that logic feels natural because it’s how we’re used to thinking, sequentially. `getUser then getWeather then update the UI with the data`.

------

Now it’s clear that promises drastically increase the readability of our asynchronous code, but is there a way we can make it even better? Assume that you were on the TC39 committee and you had all the power to add new features to the JavaScript language. What steps, if any, would you take to improve this code?

```js
$("#btn").on("click", () => {
  getUser("tylermcginnis")
    .then(getWeather)
    .then((data) => updateUI(data))
    .catch(showError)
})
```

As we’ve discussed, the code reads pretty nicely. Just as our brains work, it’s in a sequential order. One issue that we did run into was that we needed to thread the data (`users`) from the first async request all the way through to the last `.then`. This wasn’t a big deal, but it made us change up our `getWeather` function to also pass along `users`. What if we just wrote our asynchronous code the same way which we write our synchronous code? If we did, that problem would go away entirely and it would still read sequentially. Here’s an idea.

```js
$("#btn").on("click", () => {
  const user = getUser('tylermcginnis')
  const weather = getWeather(user)

  updateUI({
    user,
    weather,
  })
})
```

Well, that would be nice. Our asynchronous code looks exactly like our synchronous code. There’s no extra steps our brain needs to take because we’re already very familiar with this way of thinking. Sadly, this obviously won’t work. As you know, if we were to run the code above, `user` and `weather` would both just be promises since that’s what `getUser` and `getWeather` return. But remember, we’re on TC39. We have all the power to add any feature to the language we want. As is, this code would be really tricky to make work. We’d have to somehow teach the JavaScript engine to know the difference between asynchronous function invocations and regular, synchronous function invocations on the fly. Let’s add a few keywords to our code to make it easier on the engine.

First, let’s add a keyword to the main function itself. This could clue the engine to the fact that inside of this function, we’re going to have some asynchronous function invocations. Let’s use `async` for this.

```js
$("#btn").on("click", async () => {
  const user = getUser('tylermcginnis')
  const weather = getWeather(user)

  updateUI({
    user,
    weather,
  })
})
```

Cool. That seems reasonable. Next let’s add another keyword to let the engine know exactly when a function being invoked is asynchronous and is going to return a promise. Let’s use `await`. As in, “Hey engine. This function is asynchronous and returns a promise. Instead of continuing on like you typically do, go ahead and ‘await’ the eventual value of the promise and return it before continuing”. With both of our new `async` and `await` keywords in play, our new code will look like this.

```js
$("#btn").on("click", async () => {
  const user = await getUser('tylermcginnis')
  const weather = await getWeather(user.location)

  updateUI({
    user,
    weather,
  })
})
```

Pretty slick. We’ve invented a reasonable way to have our asynchronous code look and behave as if it were synchronous. Now the next step is to actually convince someone on TC39 that this is a good idea. Lucky for us, as you probably guessed by now, we don’t need to do any convincing because this feature is already part of JavaScript and it’s called `Async/Await`.

Don’t believe me? [Here’s our live code now](https://codesandbox.io/s/00w10o19xn) that we’ve added Async/Await to it. Feel free to play around with it.

------

#### async functions return a promise

Now that you’ve seen the benefit of Async/Await, let’s discuss some smaller details that are important to know. First, anytime you add `async` to a function, that function is going to implicitly return a promise.

```js
async function getPromise(){}

const promise = getPromise()
```

Even though `getPromise` is literally empty, it’ll still return a promise since it was an `async` function.

If the `async` function returns a value, that value will also get wrapped in a promise. That means you’ll have to use `.then` to access it.

```js
async function add (x, y) {
  return x + y
}

add(2,3).then((result) => {
  console.log(result) // 5
})
```

------

#### await without async is bad

If you try to use the `await` keyword inside of a function that isn’t `async`, you’ll get an error.

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

Here’s how I think about it. When you add `async` to a function it does two things. It makes it so the function itself returns (or wraps what gets returned in) a promise and makes it so you can use `await` inside of it.

------

#### Error Handling

You may have noticed we cheated a little bit. In our original code we had a way to catch any errors using `.catch`. When we switched to Async/Await, we removed that code. With Async/Await, the most common approach is to wrap your code in a `try/catch` block to be able to catch the error.

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