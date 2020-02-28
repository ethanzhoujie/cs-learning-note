## Shorthand Syntax

ES6 introduced two new features to make objects more concise - Shorthand Properties and Shorthand Method Names.

### Shorthand Properties

With Shorthand Properties, whenever you have a variable which is the same name as a property on an object, when constructuring the object, you can omit the property name.

What that means is that code that used to look like this,

```js
function formatMessage (name, id, avatar) {
  return {
    name: name,    id: id,    avatar: avatar,    timestamp: Date.now()
  }
}
```

can now look like this.

```js
function formatMessage (name, id, avatar) {
  return {
    name,    id,    avatar,    timestamp: Date.now()
  }
}
```

### Shorthand Method Names

Now, what if one of those properties was a function?

A function that is a property on an object is called a method. With ES6’s Shorthand Method Names, you can omit the `function` keyword completely. What that means is that code that used to look like this,

```js
function formatMessage (name, id, avatar) {
  return {
    name,
    id,
    avatar,
    timestamp: Date.now(),
    save: function () {      // save message    }  }
}
```

can now look like this

```js
function formatMessage (name, id, avatar) {
  return {
    name,
    id,
    avatar,
    timestamp: Date.now(),
    save () {      //save message    }  }
}
```

------

Both Shorthand Properties and Shorthand Method Names are just syntactical sugar over the previous ways we used to add properties to an object. However, because they’re such common tasks, even the smallest improvements eventually add up.