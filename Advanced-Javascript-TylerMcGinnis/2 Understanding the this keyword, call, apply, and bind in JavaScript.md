## Understanding the this keyword, call, apply, and bind in JavaScript

**The “this” keyword allows you to decide which object should be focal when invoking a function or a method.** 

Now that you know the first step to figuring out what the `this` keyword is referencing is to look at where the function is being invoked, 

what’s next? To help us with the next step, we’re going to establish 5 rules or guidelines.

1. Implicit Binding
2. Explicit Binding
3. new Binding
4. Lexical Binding
5. window Binding

### Implicit Binding

**Look to the left of the dot when the function is invoked**: If there is a “dot”, look to the left of that dot to find the object that the `this` keyword is referencing.

```js
const user = {
  name: 'Tyler',
  age: 27,
  greet() {
    alert(`Hello, my name is ${this.name}`)
  },
  mother: {
    name: 'Stacey',
    greet() {
      alert(`Hello, my name is ${this.name}`)
    }
  }
}

user.greet() // Tyler
user.mother.greet() // Stacey 
```

### Explicit Binding

> “call”, “apply” and “bind” are the methods on every function that allows you to invoke the function specifying in what context the function will be invoked. The first argument you pass to it will be the context (or the focal object)

```js
// The first argument is the context
greet.call(user) 
greet.apply(user)
const newFn = greet.bind(user)
```

Now to pass arguments to a function being invoked with `.call`, you pass them in one by one after you specify the first argument which is the context.

```js
function greet (l1, l2, l3) {
  alert(
    `Hello, my name is ${this.name} and I know ${l1}, ${l2}, and ${l3}`
  )
}

const user = {
  name: 'Tyler',
  age: 27,
}

const languages = ['JavaScript', 'Ruby', 'Python']

// With call, pass in parameters one by one
greet.call(user, languages[0], languages[1], languages[2])
// With apply, pass in parameters as an array
greet.apply(user, languages)
// With bind, it's not immediately invoking the function, it'll return a new function  // that you can invoke at a later time 
const newFn = greet.bind(user, languages[0], languages[1], languages[2])
newFn() // alerts "Hello, my name is Tyler and I know JavaScript, Ruby, and Python"
```

### new Binding

The third rule for figuring out what the `this` keyword is referencing is called the `new` binding. 

Whenever you invoke a function with the `new` keyword, under the hood, the JavaScript interpreter will create a brand new object for you and call it `this`. 

So, naturally, if a function was called with `new`, the `this` keyword is referencing that new object that the interpreter created.

```js
function User (name, age) {
  /*
    Under the hood, JavaScript creates a new object called `this`
    which delegates to the User's prototype on failed lookups. If a
    function is called with the new keyword, then it's this new object that interpreter created that the this keyword is referencing.
  */ 

  this.name = name
  this.age = age
}

const me = new User('Tyler', 27)
```

### Lexical Binding

At this point, we’re on our 4th rule and you may be feeling a bit overwhelmed. That’s fair. The `this` keyword in JavaScript is arguably more complex than it should be. Here’s the good news, this next rule is the most intuitive.

Odds are you’ve heard of and used an arrow function before. They’re new as of ES6. They allow you to write functions in a more concise format.

```js
friends.map((friend) => friend.name)
```

Even more than conciseness, arrow functions have a much more intuitive approach when it comes to `this`keyword. Unlike normal functions, arrow functions don’t have their own `this`. Instead, `this` is determined `lexically`. That’s a fancy way of saying `this` is determined how you’d expect, following the normal variable lookup rules. Let’s continue with the example we used earlier. Now, instead of having `languages` and `greet` as separate from the object, let’s combine them.

```js
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {}
}
```

Earlier we assumed that the `languages` array would always have a length of 3. By doing so we were able to use hardcoded variables like `l1`, `l2`, and `l3`. Let’s make `greet` a little more intelligent now and assume that `languages` can be of any length. To do this, we’ll use `.reduce` in order to create our string.

```js
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`

    const langs = this.languages.reduce(function (str, lang, i) {
      if (i === this.languages.length - 1) {
        return `${str} and ${lang}.`
      }

      return `${str} ${lang},`
    }, "")

    alert(hello + langs)
  }
}
```

That’s a lot more code but the end result should be the same. When we invoke `user.greet()`, we expect to see `Hello, my name is Tyler and I know JavaScript, Ruby, and Python.`. Sadly, there’s an error. Can you spot it? Grab the code above and run it in your console. You’ll notice it’s throwing the error `Uncaught TypeError: Cannot read property 'length' of undefined`. Gross. The only place we’re using `.length` is on line 9, so we know our error is there.

```js
if (i === this.languages.length - 1) {}
```

According to our error, `this.langauges` is undefined. Let’s walk through our steps to figure out what that `this`keyword is referencing cause clearly, it’s not referencing `user` at it should be. First, we need to look at where the function is being invoked. Wait? Where is the function being invoked? The function is being passed to `.reduce` so we have no idea. We never actually see the invocation of our anonymous function since JavaScript does that itself in the implementation of `.reduce`. That’s the problem. We need to specify that we want the anonymous function we pass to `.reduce` to be invoked in the context of `user`. That way `this.languages` will reference `user.languages`. As we learned above, we can use `.bind`.

```js
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`

    const langs = this.languages.reduce(function (str, lang, i) {
      if (i === this.languages.length - 1) {
        return `${str} and ${lang}.`
      }

      return `${str} ${lang},`
    }.bind(this), "")

    alert(hello + langs)
  }
}
```

So we’ve seen how `.bind` solves the issue, but what does this have to do with arrow functions. Earlier I said that with arrow functions "`this` is determined `lexically`. That’s a fancy way of saying `this` is determined how you’d expect, following the normal variable lookup rules."

In the code above, following just your natural intuition, what would the `this` keyword reference inside of the anonymous function? For me, it should reference `user`. There’s no reason to create a new context just because I had to pass a new function to `.reduce`. And with that intuition comes the often overlooked value of arrow functions. If we re-write the code above and do nothing but use an anonymous arrow function instead of an anonymous function declaration, everything “just works”.

```js
const user = {
  name: 'Tyler',
  age: 27,
  languages: ['JavaScript', 'Ruby', 'Python'],
  greet() {
    const hello = `Hello, my name is ${this.name} and I know`

    const langs = this.languages.reduce((str, lang, i) => {
      if (i === this.languages.length - 1) {
        return `${str} and ${lang}.`
      }

      return `${str} ${lang},`
    }, "")

    alert(hello + langs)
  }
}
```

Again the reason for this because with arrow functions, `this` is determined “lexically”. Arrow functions don’t have their own `this`. Instead, just like with variable lookups, the JavaScript interpreter will look to the enclosing (parent) scope to determine what `this` is referencing.

### window Binding

Finally is the “catch-all” case - the window binding. Let’s say we had the following code

```js
function sayAge () {
  console.log(`My age is ${this.age}`)
}

const user = {
  name: 'Tyler',
  age: 27
}
```

As we covered earlier, if you wanted to invoke `sayAge` in the context of `user`, you could use `.call`, `.apply`, or `.bind`. What would happen if we didn’t use any of those and instead just invoked `sayAge` as you normally would

```js
sayAge() // My age is undefined
```

What you’d get is, unsurprisingly, `My age is undefined` because `this.age` would be undefined. Here’s where things get a little weird. What’s really happening here is because there’s nothing to the left of the dot, we’re not using `.call`, `.apply`, `.bind`, or the `new` keyword, JavaScript is defaulting `this` to reference the `window`object. What that means is if we add an `age` property to the `window` object, then when we invoke our `sayAge`function again, `this.age` will no longer be undefined but instead, it’ll be whatever the `age` property is on the window object. Don’t believe me? Run this code,

```js
window.age = 27

function sayAge () {
  console.log(`My age is ${this.age}`)
}
```

Pretty gnarly, right? That’s why the 5th rule is the `window Binding`. If none of the other rules are met, then JavaScript will default the `this` keyword to reference the `window` object.

------

> As of ES5, if you have “strict mode” enabled, JavaScript will do the right thing and instead of defaulting to the window object will just keep “this” as undefined.

```js
'use strict'

window.age = 27

function sayAge () {
  console.log(`My age is ${this.age}`)
}

sayAge() // TypeError: Cannot read property 'age' of undefined
```

------

So putting all of our rules into practice, whenever I see the `this` keyword inside of a function, these are the steps I take in order to figure out what it’s referencing.

1. Look to where the function was invoked.
2. Is there an object to the left of the dot? If so, that’s what the “this” keyword is referencing. If not, continue to #3.
3. Was the function invoked with “call”, “apply”, or “bind”? If so, it’ll explicitly state what the “this” keyword is referencing. If not, continue to #4.
4. Was the function invoked using the “new” keyword? If so, the “this” keyword is referencing the newly created object that was made by the JavaScript interpreter. If not, continue to #5.
5. Is “this” inside of an arrow function? If so, its reference may be found lexically in the enclosing (parent) scope. If not, continue to #6.
6. Are you in “strict mode”? If yes, the “this” keyword is undefined. If not, continue to #6.
7. JavaScript is weird. “this” is referencing the “window” object.