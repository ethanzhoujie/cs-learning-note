## Private and Public Class Fields

My favorite part of the JavaScript community is that everyone seems to always be asking *“why?”*. Why do we do things the way we do them? Generally, the answer to that question is full of reason and historical context. But sometimes, the answer tends to be more simple - “because that’s what we’ve always done.”

Here’s a simple `Player` class.

```js
class Player {
  constructor() {
    this.points = 0
    this.assists = 0
    this.rebounds = 0
    this.steals = 0
  }
  addPoints(amount) {
    this.points += amount
  }
  addAssist() {
    this.assists++
  }
  addRebound() {
    this.rebounds++
  }
  addSteal() {
    this.steals++
  }
}
```

Looking at that code, is there any way we can make it a little more intuitive? The methods are fine, those come pretty naturally. What about the constructor? What even is a `constructor` and why do we have to define instance values there? Now, there are answers to those questions but why can’t we just add state to our instances just like we did with the methods? Something like this

```js
class Player {
  points = 0
  assists = 0
  rebounds = 0
  steals = 0
  addPoints(amount) {
    this.points += amount
  }
  addAssist() {
    this.assists++
  }
  addRebound() {
    this.rebounds++
  }
  addSteal() {
    this.steals++
  }
}
```

It turns out this is the foundation for the [Class Fields Declaration](https://github.com/tc39/proposal-class-fields) proposal which is currently at Stage 3 in the TC-39 process. This proposal will allow you to add instance properties directly as a property on the class without having to use the `constructor` method. Pretty slick, but where this proposal really shines is if we look at some React code. Here’s a typical React component. It has local state, some methods, and a few static properties being added to the class.

```js
class PlayerInput extends Component {
  constructor(props) {
    super(props)
    this.state = {
      username: ''
    }

    this.handleChange = this.handleChange.bind(this)
  }
  handleChange(event) {
    this.setState({
      username: event.target.value
    })
  }
  render() {
    ...
  }
}

PlayerInput.propTypes = {
  id: PropTypes.string.isRequired,
  label: PropTypes.string.isRequired,
  onSubmit: PropTypes.func.isRequired,
}

PlayerInput.defaultProps = {
  label: 'Username',
}
```

Let’s see how the new `Class Fields` proposal improves the code above First, we can take our `state` variable out of the constructor and define it directly as a property (or “field”) on the class.

```js
class PlayerInput extends Component {
  state = {
    username: ''
  }
  constructor(props) {
    super(props)

    this.handleChange = this.handleChange.bind(this)
  }
  handleChange(event) {
    this.setState({
      username: event.target.value
    })
  }
  render() {
    ...
  }
}

PlayerInput.propTypes = {
  id: PropTypes.string.isRequired,
  label: PropTypes.string.isRequired,
  onSubmit: PropTypes.func.isRequired,
}

PlayerInput.defaultProps = {
  label: 'Username',
}
```

Cool, but nothing to get too excited over. Let’s keep going. In the previous post, we talked about how you can add static methods to the class itself by using the `static` keyword. However, according to the ES6 class specification, this only works with methods, not values. That’s why in the code above we have to add `propTypes` and `defaultProps` to `PlayerInput` after we define it and not in the class body. Again, why can’t those go directly on the class body just as a static method would? Well, the good news is this is encompassed in the `Class Fields` proposal as well. So now instead of just defining static methods in the class body, you can also define static values. What that means for our code is we can move `propTypes` and `defaultProps` up into the class definition.

```js
class PlayerInput extends Component {
  static propTypes = {
    id: PropTypes.string.isRequired,
    label: PropTypes.string.isRequired,
    onSubmit: PropTypes.func.isRequired,
  }
  static defaultProps = {
    label: 'Username'
  }
  state = {
    username: ''
  }
  constructor(props) {
    super(props)

    this.handleChange = this.handleChange.bind(this)
  }
  handleChange(event) {
    this.setState({
      username: event.target.value
    })
  }
  render() {
    ...
  }
}
```

Much better, but we still have that ugly `constructor` method and `super` invocation. Again, the reason we need the constructor right now is in order to bind the `handleChange` method to the correct context. If we could figure out another way to make sure `handleChange` was always invoked in the correct context, we could get rid of the `constructor` altogether.

If you’ve used arrow functions before, you know that they don’t have their own `this` keyword. Instead, the `this` keyword is bound `lexically`. That’s a fancy way of saying when you use the `this` keyword inside of an arrow function, things behave how you’d expect them to. Taking that knowledge and combining it with the “Class Fields” proposal, what if we swapped out the `handleChange` method for an arrow function? Seems a little weird but by doing this we’d get rid of the `.bind` issue altogether since, again, arrow functions bind `this` lexically.

```js
class PlayerInput extends Component {
  static propTypes = {
    id: PropTypes.string.isRequired,
    label: PropTypes.string.isRequired,
    onSubmit: PropTypes.func.isRequired,
  }
  static defaultProps = {
    label: 'Username'
  }
  state = {
    username: ''
  }
  handleChange = (event) => {
    this.setState({
      username: event.target.value
    })
  }
  render() {
    ...
  }
}
```

Well, would you look at that? That’s **much** better than the original class we started with and it’s all thanks to the Class Fields proposal which will be part of the official EcmaScript specification soon.

From a developer experience standpoint, Class Fields are a clear win. However, there are some downsides to them that are rarely talked about. In the last post, we talked about how ES6 classes are just sugar over what we called the “pseudo-classical” pattern. Meaning, when you add a method to a class, that’s really like adding a method to the function’s prototype.

```js
class Animal {
  eat() {}
}

// Is equivalent to

function Animal () {}
Animal.prototype.eat = function () {}
```

This is performant because `eat` is defined once and shared across all instances of the class. What does this have to do with Class Fields? Well, as we saw above, Class Fields are added to the instance. This means that for each instance we create, we’ll be creating a new `eat` method.

```js
class Animal {
  eat() {}
  sleep = () => {}
}

// Is equivalent to

function Animal () {
  this.sleep = function () {}
}

Animal.prototype.eat = function () {}
```

Notice how `sleep` gets put on the instance and not on `Animal.prototype`. Is this a bad thing? Well, it can be. Making broad statements about performance without measuring is generally a bad idea. The question you need to answer in your application is if the developer experience you gain from Class Fields outweighs the potential performance hit.

> If you want to use any of what we’ve talked about so far in your app, you’ll need to use the [babel-plugin-transform-class-properties](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/) plugin.

## Private Fields

Another aspect of the Class Fields proposal are “private fields”. Sometimes when you’re building a class, you want to have private values that aren’t exposed to the outside world. Historically in JavaScript, because we’ve lacked the ability to have truly private values, we’ve marked them with an underscore.

```js
class Car {
  _milesDriven = 0
  drive(distance) {
    this._milesDriven += distance
  }
  getMilesDriven() {
    return this._milesDriven
  }
}
```

In the example above, we’re relying on the consumer of the `Car` class to get the car’s mileage by invoking the `getMilesDriven` method. However, because there’s really nothing making `_milesDriven` private, any instance can access it.

```js
const tesla = new Car()
tesla.drive(10)
console.log(tesla._milesDriven)
```

There are fancy (hacky) ways around this problem using WeakMaps, but it would be nice if a simpler solution existed. Again, the Class Fields proposal is coming to our rescue. According to the proposal, you can create a private field using a **#**. Yes, you read that right, **#**. Let’s take a look at what that does to our code,

```js
class Car {
  #milesDriven = 0
  drive(distance) {
    this.#milesDriven += distance
  }
  getMilesDriven() {
    return this.#milesDriven
  }
}
```

and we can go one step further with the shorthand syntax

```js
class Car {
  #milesDriven = 0
  drive(distance) {
    #milesDriven += distance
  }
  getMilesDriven() {
    return #milesDriven
  }
}

const tesla = new Car()
tesla.drive(10)
tesla.getMilesDriven() // 10
tesla.#milesDriven // Invalid
```

If you’re interested in more of the details/decisions behind private fields, there’s a great [write-up here](https://github.com/tc39/proposal-private-fields/blob/master/FAQ.md).

> There’s [currently a PR](https://github.com/babel/proposals/issues/12) to add private fields to Babel so you can use them in your apps.