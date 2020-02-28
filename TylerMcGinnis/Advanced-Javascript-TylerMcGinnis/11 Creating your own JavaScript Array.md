## Creating your own JavaScript Array

When you’re first learning anything new, it’s hard to see the bigger picture. Generally, your focus is on how to **use** the thing rather than how the thing **works**. Take a car for example. When you first start driving, you’re not worried about how the engine works. Instead, you’re just trying not to crash and die.

When you first started with JavaScript, odds are one of the first data structures you learned was an array. Your concern was most likely memorizing the array API and how you’d use it, not how it actually works. Since that day, have you ever taken a step back and **really** thought about how arrays work? Probably not, and that’s fine. But today all of that is going to change. **The goal here is to take the knowledge and patterns you’ve learned in this course and to use them to re-create a small portion of the JavaScript array API.**

Here’s the result we’re going for.

```js
const friends = array('Jordyn', 'Mikenzi')

friends.push('Joshy') // 3
friends.push('Jake') // 4

friends.pop() // Jake

friends.filter((friend) => friend.charAt(0) !== 'J') // ['Mikenzi']

friends // { 0: 'Jordyn', 1: 'Mikenzi', 2: 'Joshy', length: 3, push: fn, pop: fn, filter: fn }
```

------

We first need to think about what an Array in JavaScript actually is. The good news is we don’t need to think too hard since we can use JavaScript’s `typeof` operator.

```js
const arr = []
typeof arr // "object"
```

It turns out an array was just an object all along 🌈. An array is just an object with numerical keys and a length property that’s managed automatically for you. Instead of manually adding or removing values from the object, you do it via the array API, `.push`, `.pop`, etc. This becomes even more clear when you look at how you use bracket notation on both objects and arrays to access values.

```js
const friendsArray = ['Jake', 'Jordyn', 'Mikenzi']
const friendsObj = {0: 'Jake', 1: 'Jordyn', 2: 'Mikenzi'}

friendsArray[1] // Jordyn
friendsObj[1] // Jordyn
```

It’s a little weird to have an object with numerical keys (since that’s literally what an array is for), but it paints a good picture that arrays really are just fancy objects. With this in mind, we can take the first step for creating our `array` function. `array` needs to return an object with a length property that delegates to `array.prototype` (since that’s where we’ll be putting all the methods). As we’ve done in previous sections, we can use `Object.create` for this.

```js
function array () {
  let arr = Object.create(array.prototype)
  arr.length = 0

  return arr
}
```

That’s a good start. Since we’re using Object.create to delegate failed lookups to `array.prototype`, we can now add any methods we want shared across all instances to `array.prototype`. If that’s still a little fuzzy, read the previous section on “A Beginner’s Guide to JavaScript’s Prototype”

Now before we move onto the methods, we first need to have our `array` function accept n amount of arguments and add those as numerical properties onto the object. We could use JavaScript’s spread operator to turn `arguments` into an array, but that feels like cheating since we’re pretending we’re re-creating arrays. Instead, we’ll use a trusty `for in` loop to loop over `arguments` and add the keys/values to our array and increment `length`.

```js
function array () {
  let arr = Object.create(array.prototype)
  arr.length = 0

  for (key in arguments) {
    arr[key] = arguments[key]
    arr.length += 1
  }

  return arr
}

const friends = array('Jake', 'Mikenzi', 'Jordyn')
friends[0] // Jake
friends[2] // Jordyn
friends.length // 3
```

So far, so good. We have the foundation for our `array` function.

Now as we saw above, we’re going to implement three different methods, `push`, `pop`, and `filter`. Since we want all the methods to be shared across all instances of `array`, we’re going to put them on `array.prototype`.

```js
array.prototype.push = function () {

}

array.prototype.pop = function () {

}

array.prototype.filter = function () {

}
```

Now let’s implement `push`. You already know what `.push` does, but how can we go about implementing it. First, we need to figure out a way to operate on whatever instance invokes `push`. This is where the `this` keyword will come into play. Inside of any of our methods, `this` is going to reference the instance which called the specific method.

```js
...

array.prototype.push = function () {
  console.log(this)
}

const friends = array('Jake', 'Jordyn', 'Mikenzi')

friends.push() // {0: "Jake", 1: "Jordyn", 2: "Mikenzi", length: 3}
```

Now that we know we can use the `this` keyword, we can start implementing `.push`. There are three things `.push` needs to do. First, it needs to add an element to our object at `this.length`, then it needs to increment `this.length` by one, and finally, it needs to return the new length of the “array”.

```js
array.prototype.push = function (element) {
  this[this.length] = element
  this.length++
  return this.length
}
```

Next, is `.pop`. `.pop` needs to do three things as well. First, it needs to remove the “last” element, or the element at `this.length - 1`. Then it needs to decrement `this.length` by one. Lastly, it needs to return the element that was removed.

```js
array.prototype.pop = function () {
  this.length--
  const elementToRemove = this[this.length]
  delete this[this.length]
  return elementToRemove
}
```

The last method we’re going to implement is `.filter`. `.filter` creates a new array after filtering out elements that don’t pass a test specified by a given function. Like we saw earlier, we can iterate over every key/value pair in the “array” by using a `for in` loop. Then for each key/value pair in the “array”, we’ll call the callback function that was passed in as the first argument. If the result of that invocation is truthy, we’ll push that into a new “array” which we’ll then return after we’ve iterated over the entire “array” instance.

```js
array.prototype.filter = function (cb) {
  let result = array()

  for (let index in this) {
    if (this.hasOwnProperty(index)) { // Avoid prototype methods
      const element = this[index]

      if (cb(element, index)) {
        result.push(element)
      }
    }
  }

  return result
}
```

At first glance, our implementation of `.filter` above looks like it should work. Spoiler alert, it doesn’t. Can you think of why it doesn’t? Here’s a hint - it has nothing to do with `.filter`. Our code for `.filter` is actually correct, it’s our `array` constructor function that is where the issue is. We can see the bug more clearly if we step through a use case for our `.filter` function.

```js
const friends = array('Jake', 'Jordyn', 'Mikenzi')

friends.filter((friend) => friend.charAt(0) !== 'J')

/* Breakdown of Iterations*/

1) friend is "Jake". The callback returns false
2) friend is "Jordyn". The callback returns false
3) friend is "Mikenzi". The callback returns true
4) friend is "length". The callback throws an error
```

Ah. We’re using a `for in` loop which by design loops over all enumerable properties of the object. In our `array` function we just set `length` by doing `this.length = 0`. That means `length` is an enumerable property and, as we saw above, will show up in `for in` loops. You may have never seen this before, but the `Object` class has a static method on it called `defineProperty` which allows you to add a property on an object and specify if that property should be `enumerable` or not. Let’s modify our `array` function to use it so we can set `length` to not be `enumerable`.

```js
function array () {
  let arr = Object.create(array.prototype)

  Object.defineProperty(arr, 'length', {
    value: 0,
    enumerable: false,
    writable: true,
  })

  for (key in arguments) {
    arr[key] = arguments[key]
    arr.length += 1
  }

  return arr
}
```

Perfect.

------

Here is all of our code altogether, including our example use cases from the beginning of the article.

```js
function array () {
  let arr = Object.create(array.prototype)

  Object.defineProperty(arr, 'length', {
    value: 0,
    enumerable: false,
    writable: true,
  })

  for (key in arguments) {
    arr[key] = arguments[key]
    arr.length += 1
  }

  return arr
}

array.prototype.push = function (element) {
  this[this.length] = element
  this.length++
  return this.length
}

array.prototype.pop = function () {
  this.length--
  const elementToRemove = this[this.length]
  delete this[this.length]
  return elementToRemove
}

array.prototype.filter = function (cb) {
  let result = array()

  for (let index in this) {
    if (this.hasOwnProperty(index)) {
      const element = this[index]

      if (cb(element, index)) {
        result.push(element)
      }
    }
  }

  return result
}

let friends = array('Jordyn', 'Mikenzi')

friends.push('Joshy') // 3
friends.push('Jake') // 4

friends.pop() // Jake

friends.filter((friend) => friend.charAt(0) !== 'J') // { 0: "Mikenzi", length: 1 }
```

------

Nice work! Even though this exercise doesn’t have any practical value, I hope it’s helped you understand a little bit more about the JavaScript language.