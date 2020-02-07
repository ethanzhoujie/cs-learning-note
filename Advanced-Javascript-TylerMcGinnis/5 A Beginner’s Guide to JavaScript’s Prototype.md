## A Beginner’s Guide to JavaScript’s Prototype

#### Functional Instantiation

```javascript
function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy

  animal.eat = function (amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }

  animal.sleep = function (length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }

  animal.play = function (length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
```

Waste memory and making each animal object bigger than it needs to be.



#### Functional Instantiation with Shared Methods

```javascript
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy
  animal.eat = animalMethods.eat
  animal.sleep = animalMethods.sleep
  animal.play = animalMethods.play

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
```



#### Object.create

**Object.create allows you to create an object which will delegate to another object on failed lookups**. 

Object.create allows you to create an object and whenever there’s a failed property lookup on that object, it can consult another object to see if that other object has the property.

```javascript
const parent = {
  name: 'Stacey',
  age: 35,
  heritage: 'Irish'
}

const child = Object.create(parent)
child.name = 'Ryan'
child.age = 7

console.log(child.name) // Ryan
console.log(child.age) // 7
console.log(child.heritage) // Irish
```



#### Functional Instantiation with Shared Methods and Object.create

```javascript
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
  let animal = Object.create(animalMethods)
  animal.name = name
  animal.energy = energy

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)

leo.eat(10)
snoop.play(5)
```



#### Prototypal Instantiation

```javascript
function Animal (name, energy) {
  let animal = Object.create(Animal.prototype)
  animal.name = name
  animal.energy = energy

  return animal
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

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)

leo.eat(10)
snoop.play(5)
```



#### Pseudoclassical Instantiation

Here’s the cool thing about `new` - when you invoke a function using the `new` keyword, those two lines are done for you implicitly (“under the hood”) and the object that is created is called `this`.

```javascript
function Animal (name, energy) {
  // const this = Object.create(Animal.prototype)

  this.name = name
  this.energy = energy

  // return this
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```



#### New Class Instantiation

```javascript
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```



Whenever you have a method that is specific to a class itself but doesn’t need to be shared across instances of that class, you can add it as a `static` property of the class.

```javascript
class Animal {
  constructor(name, energy) {
    this.name = name
    this.energy = energy
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
  static nextToEat(animals) {
    const sortedByLeastEnergy = animals.sort((a,b) => {
      return a.energy - b.energy
    })

    return sortedByLeastEnergy[0].name
  }
}
```

With ES5, this same pattern is as simple as just manually adding the method to the function object instead of in the prototype.

```javascript
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

Animal.nextToEat = function (nextToEat) {
  const sortedByLeastEnergy = animals.sort((a,b) => {
    return a.energy - b.energy
  })

  return sortedByLeastEnergy[0].name
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)

console.log(Animal.nextToEat([leo, snoop])) // Leo
```



### Getting the prototype of an object

```javascript
...
const prototype = Object.getPrototypeOf(leo)

console.log(prototype)
// {constructor: ƒ, eat: ƒ, sleep: ƒ, play: ƒ}

prototype === Animal.prototype // true
```



### Determining if a property lives on the prototype

```javascript
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

for(let key in leo) {
  console.log(`Key: ${key}. Value: ${leo[key]}`)
}
```

```markd
Key: name. Value: Leo
Key: energy. Value: 7
Key: eat. Value: function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}
Key: sleep. Value: function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}
Key: play. Value: function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}
```

```javascript
...

const leo = new Animal('Leo', 7)

for(let key in leo) {
  if (leo.hasOwnProperty(key)) {
    console.log(`Key: ${key}. Value: ${leo[key]}`)
  }
}
```

```markdown
Key: name. Value: Leo
Key: energy. Value: 7
```



### Check if an object is an instance of a Class

```javascript
object instanceof Class
```

The way that `instanceof` works is it checks for the presence of `constructor.prototype` in the object’s prototype chain. In the example above, `leo instanceof Animal` is `true` because `Object.getPrototypeOf(leo) === Animal.prototype`. In addition, `leo instanceof User` is `false` because `Object.getPrototypeOf(leo) !== User.prototype`.



### Creating new agnostic constructor functions

```javascript
function Animal (name, energy) {
  if (this instanceof Animal === false) {
    return new Animal(name, energy)
  }

  this.name = name
  this.energy = energy
}
```



### Re-creating Object.create

Throughout this post, we’ve relied heavily upon `Object.create` in order to create objects which delegate to the constructor function’s prototype. At this point, you should know how to use `Object.create` inside of your code but one thing that you might not have thought of is how `Object.create` actually works under the hood. In order for you to **really** understand how `Object.create` works, we’re going to re-create it ourselves. First, what do we know about how `Object.create` works?

1. It takes in an argument that is an object.
2. It creates an object that delegates to the argument object on failed lookups.
3. It returns the new created object.

Let’s start off with #1.

```js
Object.create = function (objToDelegateTo) {

}
```

Simple enough.

Now #2 - we need to create an object that will delegate to the argument object on failed lookups. This one is a little more tricky. To do this, we’ll use our knowledge of how the `new` keyword and prototypes work in JavaScript. First, inside the body of our `Object.create` implementation, we’ll create an empty function. Then, we’ll set the prototype of that empty function equal to the argument object. Then, in order to create a new object, we’ll invoke our empty function using the `new` keyword. If we return that newly created object, that’ll finish #3 as well.

```js
Object.create = function (objToDelegateTo) {
  function Fn(){}
  Fn.prototype = objToDelegateTo
  return new Fn()
}
```

Wild. Let’s walk through it.

When we create a new function, `Fn` in the code above, it comes with a `prototype` property. When we invoke it with the `new` keyword, we know what we’ll get back is an object that will delegate to the function’s prototype on failed lookups. If we override the function’s prototype, then we can decide which object to delegate to on failed lookups. So in our example above, we override `Fn`'s prototype with the object that was passed in when `Object.create` was invoked which we call `objToDelegateTo`.

> Note that we’re only supporting a single argument to Object.create. The official implementation also supports a second, optional argument which allows you to add more properties to the created object.



### Arrow Functions

Arrow functions don’t have their own `this` keyword. As a result, arrow functions can’t be constructor functions and if you try to invoke an arrow function with the `new` keyword, it’ll throw an error.

```js
const Animal = () => {}

const leo = new Animal() // Error: Animal is not a constructor
```

Also, because we demonstrated above that the pseudo-classical pattern can’t be used with arrow functions, arrow functions also don’t have a `prototype` property.

```js
const Animal = () => {}
console.log(Animal.prototype) // undefined
```