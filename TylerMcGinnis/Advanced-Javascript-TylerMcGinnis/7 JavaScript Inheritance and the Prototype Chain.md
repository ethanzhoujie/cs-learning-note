## JavaScript Inheritance and the Prototype Chain

Previously we learned how to create an `Animal` class both in ES5 as well as in ES6. We also learned how to share methods across those classes using JavaScript’s prototype. To review, here’s the code we saw in an earlier post.

------

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const leo = new Animal('Leo', 7)
```

------

```js
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep() {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play() {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
```

------

Now let’s say we wanted to start making individual classes for specific animals. For example, what if we wanted to start making a bunch of dog instances. What properties and methods will these dogs have? Well, similar to our `Animal` class, we could give each dog a `name`, an `energy` level, and the ability to `eat`, `sleep`, and `play`. Unique to our `Dog` class, we could also give them a `breed` property as well as the ability to `bark`. In ES5, our `Dog` class could look something like this

```js
function Dog (name, energy, breed) {
  this.name = name
  this.energy = energy
  this.breed = breed
}

Dog.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Dog.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Dog.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

Dog.prototype.bark = function () {
  console.log('Woof-Woof!')
  this.energy -= .1
}

const charlie = new Dog('Charlie', 10, 'Goldendoodle')
```

Alright, well… we just recreated the `Animal` class and added a few new properties to it. If we wanted to create another animal, say a `Cat`, at this point we’d again have to create a `Cat` class, duplicate all the common logic located in the `Animal` class to it, then add on `Cat` specific properties just like we did with the `Dog` class. In fact, we’d have to do this for each different type of animal we created.

```js
function Dog (name, energy, breed) {}

function Cat (name, energy, declawed) {}

function Giraffe (name, energy, height) {}

function Monkey (name, energy, domesticated) {}
```

This work, but it seems wasteful. The `Animal` class is the perfect base class. What that means is that it has all the properties that each one of our animals has in common. Whether we’re creating a dog, cat, giraffe, or monkey, all of them will have a `name`, `energy` level, and the ability to `eat`, `sleep`, and `play`. With that said, is there a way we can utilize the `Animal` class whenever we create the individual classes for each different animal? Let’s try it out. I’ll paste the `Animal` class again below for easy reference.

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

function Dog (name, energy, breed) {

}
```

What are some things we know about the `Dog` constructor function above?

First, we know it takes 3 arguments, `name`, `energy`, and `breed`.

Second, we know it’s going to be called with the `new` keyword so we’ll have a `this` object.

And third, we know we need to utilize the `Animal` function so that any instance of dog will have a `name`, `energy` level, and be able to `eat`, `sleep`, and `play`.

It’s the third one that’s the tricky one. The way you “utilize” a function is by calling it. So we know that inside of `Dog`, we want to call `Animal`. What we need to figure out though is how we can invoke `Animal` in the context of `Dog`. What that means is that we want to call `Animal` with the `this` keyword from `Dog`. If we do that correctly, then `this` inside of the `Dog` function will have all the properties of `Animal` (`name`, `energy`). If you remember from a [previous section](https://tylermcginnis.com/this-keyword-call-apply-bind-javascript/), every function in JavaScript has a `.call` method on it.

> .call() is a method on every function that allows you to invoke the function specifying in what context the function will be invoked.

This sounds like exactly what we need. We want to invoke `Animal` in the context of `Dog`.

```js
function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

const charlie = new Dog('Charlie', 10, 'Goldendoodle')

charlie.name // Charlie
charlie.energy // 10
charlie.breed // Goldendoodle
```

Solid, we’re half-way there. You’ll notice in the code above that because of this line `Animal.call(this, name, energy)`, every instance of `Dog` will now have a `name` and `energy` property. Again, the reason for that is because it’s as if we ran the `Animal` function with the `this` keyword generated from `Dog`. Then after we added a `name` and `energy` property to `this`, we also added a `breed` property just as we normally would.

Remember the goal here is to have each instance of `Dog` have not only all the properties of `Animal`, but also all the methods as well. If you run the code above, you’ll notice that if you try to run `charlie.eat(10)` you’ll get an error. Currently, every instance of `Dog` will have the properties of `Animal` (`name` and `energy`), but we haven’t done anything to make sure that they also have the methods (`play`, `eat`, `sleep`).

Let’s think about how we can solve this. We know that all the `Animal`'s methods are located on `Animal.prototype`. What that means is we somehow want to make sure that all instances of `Dog` will have access to the methods on `Animal.prototype`. What if we used our good friend `Object.create` here? If you’ll remember, `Object.create` allows you to create an object which will delegate to another object on failed lookups. So in our case, the object we want to create is going to be `Dog`'s prototype and the object we want to delegate to on failed lookups is `Animal.prototype`.

```js
function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)
```

Now, whenever there’s a failed lookup on an instance of `Dog`, JavaScript will delegate that lookup to `Animal.prototype`. If this is still a little fuzzy, re-read [A Beginner’s Guide to JavaScript’s Prototype](http://tylermcginnis.com/beginners-guide-to-javascript-prototype/) where we talk all about `Object.create` and JavaScript’s prototype.

Let’s look at the full code together then we’ll walk through what happens.

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)
```

Now we’ve created our base class (`Animal`) as well as our subclass (`Dog`), let’s see what it looks like under the hood when we create an instance of `Dog`.

```js
const charlie = new Dog('Charlie', 10, 'Goldendoodle')

charlie.name // Charlie
charlie.energy // 10
charlie.breed // Goldendoodle
```

Nothing fancy so far, but let’s look at what happens when we invoke a method located on `Animal`.

```js
charlie.eat(10)

/*
1) JavaScript checks if charlie has an eat property - it doesn't.
2) JavaScript then checks if Dog.prototype has an eat property
    - it doesn't.
3) JavaScript then checks if Animal.prototype has an eat property
    - it does so it calls it.
*/
```

The reason `Dog.prototype` gets checked is because when we created a new instance of `Dog`, we used the `new` keyword. Under the hood, the `this` object that was created for us delegates to `Dog.prototype` (seen in comments below).

```js
function Dog (name, energy, breed) {
  // this = Object.create(Dog.prototype)
  Animal.call(this, name, energy)

  this.breed = breed
  // return this
}
```

The reason `Animal.prototype` gets checked is because we overwrote `Dog.prototype` to delegate to `Animal.prototype` on failed lookups with this line

```js
Dog.prototype = Object.create(Animal.prototype)
```

Now one thing we haven’t talked about is what if `Dog` has its own methods? Well, that’s a simple solution. Just like with `Animal`, if we want to share a method across all instances of that class, we add it to the function’s prototype.

```js
...

function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.bark = function () {
  console.log('Woof Woof!')
  this.energy -= .1
}
```

👌 very nice. There’s just one small addition we need to make. If you remember back to the [Beginner’s Guide to JavaScript’s Prototype](http://tylermcginnis.com/beginners-guide-to-javascript-prototype/) post, we were able to get access to the instances’ constructor function by using `instance.constructor`.

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

const leo = new Animal('Leo', 7)
console.log(leo.constructor) // Logs the constructor function
```

As explained in the previous post, “the reason this works is because any instances of `Animal` are going to delegate to `Animal.prototype` on failed lookups. So when you try to access `leo.constructor`, `leo` doesn’t have a `constructor` property so it will delegate that lookup to `Animal.prototype` which indeed does have a `constructor` property.”

The reason I bring this up is because in our implementation, we overwrote `Dog.prototype` with an object that delegates to `Animal.prototype`.

```js
function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.bark = function () {
  console.log('Woof Woof!')
  this.energy -= .1
}
```

What that means is that now, any instances of `Dog` which log `instance.constructor` are going to get the `Animal` constructor rather than the `Dog` constructor. You can see for yourself by running this code -

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.bark = function () {
  console.log('Woof Woof!')
  this.energy -= .1
}

const charlie = new Dog('Charlie', 10, 'Goldendoodle')
console.log(charlie.constructor)
```

Notice it gives you the `Animal` constructor even though `charlie` is a direct instance of `Dog`. Again, we can walk through what’s happening here just like we did above.

```js
const charlie = new Dog('Charlie', 10, 'Goldendoodle')
console.log(charlie.constructor)

/*
1) JavaScript checks if charlie has a constructor property - it doesn't.
2) JavaScript then checks if Dog.prototype has a constructor property
    - it doesn't because it was deleted when we overwrote Dog.prototype.
3) JavaScript then checks if Animal.prototype has a constructor property
    - it does so it logs that.
*/
```

How can we fix this? Well, it’s pretty simple. We can just add the correct `constructor` property to `Dog.prototype` once we overwrite it.

```js
function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.bark = function () {
  console.log('Woof Woof!')
  this.energy -= .1
}

Dog.prototype.constructor = Dog
```

------

At this point, if we wanted to make another subclass, say `Cat`, we’d follow the same pattern.

```js
function Cat (name, energy, declawed) {
  Animal.call(this, name, energy)

  this.declawed = declawed
}

Cat.prototype = Object.create(Animal.prototype)
Cat.prototype.constructor = Cat

Cat.prototype.meow = function () {
  console.log('Meow!')
  this.energy -= .1
}
```

This concept of having a base class with subclasses that delegate to it is called **inheritance** and it’s a staple of **Object Oriented Programming (OOP)**. If you’re coming from a different programming language, odds are you’re already familiar with OOP and inheritance. Before ES6 classes, in JavaScript, inheritance was quite the task as you can see above. You need to understand now only **when** to use inheritance, but also a nice mix of `.call`, `Object.create`, `this`, and `FN.prototype` - all pretty advanced JS topics. Let’s see how we’d accomplish the same thing using ES6 classes though.

First, let’s review what it looks like to go from an ES5 “class” to an ES6 class using our `Animal` class.

------

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const leo = new Animal('Leo', 7)
```

------

```js
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep() {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play() {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
```

------

Now that we’ve refactored our `Animal` constructor function into an ES6 class, the next thing we need to do is figure out how to refactor our base class (`Dog`). The good news is it’s much more intuitive. For reference, in ES5, here’s what we had.

```js
function Dog (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.bark = function () {
  console.log('Woof Woof!')
  this.energy -= .1
}

Dog.prototype.constructor = Dog
```

Before we get into inheritance, let’s refactor `Dog` to use an ES6 class as we learned in a previous post.

```js
class Dog {
  constructor(name, energy, breed) {
    this.breed = breed
  }
  bark() {
    console.log('Woof Woof!')
    this.energy -= .1
  }
}
```

Looks great. Now, let’s figure out how to make sure that `Dog` inherits from `Animal`. The first step we need to make is a pretty straight forward one. With ES6 classes, you can `extend` a base class with this syntax

```js
class Subclass extends Baseclass {}
```

Translated into our example, that would make our `Dog` class look like this

```js
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep() {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play() {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

class Dog extends Animal {
  constructor(name, energy, breed) {
    this.breed = breed
  }
  bark() {
    console.log('Woof Woof!')
    this.energy -= .1
  }
}
```

In ES5 in order to make sure that every instance of `Dog` had a `name` and an `energy` property, we used `.call` in order to invoke the `Animal` constructor function in the context of the `Dog` instance. Luckily for us, in ES6 it’s much more straight forward. Whenever you are extending a base class and you need to invoke that base class’ constructor function, you invoke `super` passing it any arguments it needs. So in our example, our `Dog` constructor gets refactored to look like this

```js
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep() {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play() {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

class Dog extends Animal {
  constructor(name, energy, breed) {
    super(name, energy) // calls Animal's constructor

    this.breed = breed
  }
  bark() {
    console.log('Woof Woof!')
    this.energy -= .1
  }
}
```

And that’s it. No using `.call`, no using `Object.create`, no worrying about resetting `constructor` on the prototype - just `extends` the base class and make sure to call `super`.

------

What’s interesting about JavaScript is the same patterns you’ve learned these last few posts are directly caked into the language itself. Previously you learned that the reason all instances of `Array` have access to the array methods like `pop`, `slice`, `filter`, etc. is because all of those methods live on `Array.prototype`.

```js
console.log(Array.prototype)

/*
  concat: ƒn concat()
  constructor: ƒn Array()
  copyWithin: ƒn copyWithin()
  entries: ƒn entries()
  every: ƒn every()
  fill: ƒn fill()
  filter: ƒn filter()
  find: ƒn find()
  findIndex: ƒn findIndex()
  forEach: ƒn forEach()
  includes: ƒn includes()
  indexOf: ƒn indexOf()
  join: ƒn join()
  keys: ƒn keys()
  lastIndexOf: ƒn lastIndexOf()
  length: 0n
  map: ƒn map()
  pop: ƒn pop()
  push: ƒn push()
  reduce: ƒn reduce()
  reduceRight: ƒn reduceRight()
  reverse: ƒn reverse()
  shift: ƒn shift()
  slice: ƒn slice()
  some: ƒn some()
  sort: ƒn sort()
  splice: ƒn splice()
  toLocaleString: ƒn toLocaleString()
  toString: ƒn toString()
  unshift: ƒn unshift()
  values: ƒn values()
*/
```

You also learned that the reason all instances of `Object` have access to methods like `hasOwnProperty` and `toString` is because those methods live on `Object.prototype`.

```js
console.log(Object.prototype)

/*
  constructor: ƒn Object()
  hasOwnProperty: ƒn hasOwnProperty()
  isPrototypeOf: ƒn isPrototypeOf()
  propertyIsEnumerable: ƒn propertyIsEnumerable()
  toLocaleString: ƒn toLocaleString()
  toString: ƒn toString()
  valueOf: ƒn valueOf()
*/
```

Here’s a challenge for you. With the list of Array methods and Object methods above, why does this code below work?

```js
const friends = ['Mikenzi', 'Jake', 'Ean']

friends.hasOwnProperty('push') // false
```

If you look at `Array.prototype`, there isn’t a `hasOwnProperty` method. Well if there isn’t a `hasOwnProperty` method located on `Array.prototype`, how does the `friends` array in the example above have access to `hasOwnProperty`? The reason for that is because the `Array` class extends the `Object` class. So in our example above, when JavaScript sees that `friends` doesn’t have a `hasOwnProperty` property, it checks if `Array.prototype` does. When `Array.prototype` doesn’t, it checks if `Object.prototype` does, then it invokes it. It’s the same process we’ve seen throughout this blog post.

JavaScript has two types - **Primitive** types and **Reference** types.

Primitive types are `boolean`, `number`, `string`, `null`, and `undefined` and are immutable. Everything else is a reference type and they all extend `Object.prototype`. That’s why you can add properties to functions and arrays and that’s why both functions and arrays have access to the methods located on `Object.prototype`.

```js
function speak(){}
speak.woahFunctionsAreLikeObjects = true
console.log(speak.woahFunctionsAreLikeObjects) // true

const friends = ['Mikenzi', 'Jake', 'Ean']
friends.woahArraysAreLikeObjectsToo = true
console.log(friends.woahArraysAreLikeObjectsToo) // true
```