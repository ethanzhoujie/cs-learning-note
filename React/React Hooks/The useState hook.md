Historically (with Class components), the way we’ve accomplished the state management is by adding a `state` property on the component’s instance (`this`) and updating that state with the `setState` method.

```jsx
class Theme extends React.Component {
  state = {
    theme: "light"
  }
  toDark = () => this.setState({ theme: "dark" })
  toLight = () => this.setState({ theme: "light" })
  render() {
    const { theme } = this.state

    return (
      <div className={theme}>
        {theme === "light" 
          ? <button onClick={this.toDark}>🔦</button>
          : <button onClick={this.toLight}>💡</button>}
      </div>
    )
  }
}
```

But with React Hooks, we use function components and make them stateful with the `useState` Hook.

#### useState

> 1. Add state to functional components
> 2. Preserve values between renders and trigger a re-render of the component 

`useState` comes built-in with React and can be accessed via `React.useState`.

It takes in a single argument, the initial value for that piece of state, and returns an array with the first item being the state value and the second item being a way to update that state.

```jsx
const themeArray = React.useState('light')
const theme = themeArray[0]
const setTheme = themeArray[1] 

...

theme // 'light'
setTheme('dark')
theme // 'dark'
```

The canonical and more precise way to write the code above is to use Array destructuring and put it all on one line.

```jsx
function Theme () {
  const [theme, setTheme] = React.useState('light')

  const toDark = () => setTheme('dark')
  const toLight = () => setTheme('light')

  return (
    <div className={theme}>
      {theme === "light"
        ? <button onClick={toDark}>🔦</button>
        : <button onClick={toLight}>💡</button>}
    </div>
  )
}
```

##### The Mental Model

`useState`allows you to trigger a component re-render, and it can preserve values between renders.

###### Trigger Re-renders

Similiar to `setState`. When updating the current state value with the updater function that `useState` provides, React will cause a re-render to the component, updating the UI.

###### Preserve Values

`useState` is the tool to preserve values between function calls/renders and to trigger a re-render of the component.

##### setState vs useState

First, there’s no more instance wide API for updating all of the state of the component as there was with `setState`. Instead, each piece of state comes with its own updater function. 

Second, and this is related to the first point, there’s no instance wide API for setting the initial values of all of the state properties on the component as there was with `state = {}`. Instead, each unique piece of state should have its own `useState`invocation (and therefore its own value and updater function).

- Class

```js
state = {
  loading: true,
  authed: false,
  repos: []
}
```

- useState

```js
const [ loading, setLoading ] = React.useState(true)
const [ authed, setAuthed ] = React.useState(false)
const [ repos, setRepos ] = React.useState([])
```

- State Object

Most important distinction between `setState` and `useState` is how they handle objects.

**setState**

`setState` would **merge** the two objects to form the new state.

```jsx
state = {
  loading: true,
  authed: false,
  repos: []
}
setLoading = (loading) => {
  this.setState({ 
    loading 
  }) // wouldn't modify "authed" or "repos".
}
```

**useState**

`useState` won’t merge the new object with the previous state. Instead, it’ll **replace** it completely.

```jsx
const [ state, setState ] = React.useState({
  loading: true,
  authed: false,
  repos: []
})

const setLoading = (loading) => {
  setState({
    loading
  }) // state.authed and state.repos are now gone.
}
```

Of course, you can get around this by manually merging the previous state with the new state if you want, but that’ll come with a performance hit.

```js
const setLoading = (loading) => {
  setState({
    ...state,
    loading
  })
}
```

If the most logical data type for your piece of state is an object, it’s best to use the `useReducer` Hook which we’ll see in an upcoming lesson.

##### Functional Updates

With `setState` or `useState`, whenever you set the current state based on the previous state, it’s recommended to pass a function as an argument to `setState` or `useState` instead of an object.

The reason for this is state updates may be asynchronous.

Passing them a function that receives `state` rather than relying on referencing state from the component instance.

```jsx
class Counter extends React.Component {
  state = { count: 0 }
  increment = () => this.setState(({ count }) => ({ count: count + 1}))
  decrement = () => this.setState(({ count }) => ({ count: count - 1}))
  render () {
    return (
      <React.Fragment>
        <button onClick={this.decrement}>-</button>
        <h1>{this.state.count}</h1>
        <button onClick={this.increment}>+</button>
      </React.Fragment>
    )
  }
}
```

```jsx
function Counter () {
  const [ count, setCount ] = React.useState(0)

  const increment = () => setCount((count) => count + 1)
  const decrement = () => setCount((count) => count - 1)

  return (
    <React.Fragment>
      <button onClick={decrement}>-</button>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
    </React.Fragment>
  );
}
```

##### Lazy State Initializations

Here’s a scenario. What if the initial value for a piece of state was the result of an expensive calculation? Something like this.

```jsx
function getExpensiveCount () {
  console.log('Calculating initial count')
  return 999
}

function Counter() {
  const [count, setCount] = React.useState(getExpensiveCount())

  const increment = () => setCount((count) => count + 1)
  const decrement = () => setCount((count) => count - 1)

  return (
    <React.Fragment>
      <button onClick={decrement}>-</button>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
    </React.Fragment>
  );
}
```

Even though React only uses the value calculated from `getExpensiveCount` on the initial render, anytime the component re-renders, the  `getExpensiveCount` is being invoked.

If the initial value for a piece of state is the result of an expensive calculation, you can pass `useState` a function that when invoked, will resolve to the initial state. When `useState` sees that it received a function as its initial state argument, it’ll only invoke it once on the initial render.

```jsx
function getExpensiveCount () {
  console.log('Calculating initial count')
  return 999
}

function Counter() {
  const [count, setCount] = React.useState(() => getExpensiveCount())

  const increment = () => setCount((count) => count + 1)
  const decrement = () => setCount((count) => count - 1)

  return (
    <React.Fragment>
      <button onClick={decrement}>-</button>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
    </React.Fragment>
  );
}
```

If you’re confused by the difference between the last two examples, note that the key lies in what we pass to `useState`. In the first example, we pass `useState` a function **invocation**. In the second, we pass it a function **definition**. What that means is that, from `useState`'s perspective, in the first example, it receives 999 since that’s what `getExpensiveCount` returns. In the second, it receives a function that it needs to invoke to get the initial value.