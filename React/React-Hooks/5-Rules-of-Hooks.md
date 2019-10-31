There is one rule you have to follow when using Hooks and it has to do with **where** Hooks can be called.

#### Only call Hooks from the top-level of a function component or a custom Hook.

You **can’t** call them anywhere else. That means you can’t call them from a normal function, you can’t call them from a class component, and you can’t call them anywhere that’s not on the top level like inside of a loop, if statement, or event handler.

```javascript
function Counter () {
  // 👍 from the top level function component
  const [count, setCount] = React.useState(0)

  if (count % 2 === 0) {
    // 👎 not from the top level
    React.useEffect(() => {})
  }

  const handleIncrement = () => {
    setCount((c) => c + 1)

    // 👎 not from the top level
    React.useEffect(() => {})
  }

  ...
}
```

```javascript
function useAuthed () {
  // 👍 from the top level of a custom Hook
  const [authed, setAuthed] = React.useState(false)
}
```

```javascript
class Counter extends React.Component {
  render () {
    // 👎 from inside a Class component
    const [count, setCount] = React.useState(0)
  }
}
```

```javascript
function getUser () {
  // 👎 from inside a normal function
  const [user, setUser] = React.useState(null)
}
```

The reason for this rule is because React relies on the call order of Hooks to keep track of internal state and references. If your Hooks aren’t called consistently in the same order across renders, React can’t do that.