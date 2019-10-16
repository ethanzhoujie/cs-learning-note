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

