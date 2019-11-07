#### Custom Hooks

To create a custom Hook, you simply create a function whose name starts with `use`.

Like any other Hook, your custom Hook needs to follow the rules for Hooks.

From there, because it’s just a function (and not a component), it’s not coupled to any UI and can return whatever it wants.

First, we create a function that starts with `use`.

```js
function useHover () {}
```

Then we add the logic we want to share to it.

```js
function useHover () {
  const [hovering, setHovering] = React.useState(false)
  const mouseOver = () => setHovering(true)
  const mouseOut = () => setHovering(false)}
```

Then we decide what to return. In this case, we want the consumer of `useHover` to have two pieces of data, the `hovering` state and the attributes to add to the DOM node whose `hovering` state they want to track.

```js
function useHover () {
  const [hovering, setHovering] = React.useState(false)
  const mouseOver = () => setHovering(true)
  const mouseOut = () => setHovering(false)
  const attrs = {
    onMouseOver: mouseOver, 
    onMouseOut: mouseOut
  }
  return [hovering, attrs]}
```

Now we can invoke `useHover` directly inside of any component which renders a DOM node whose `hovering`state we want to track.

```jsx
function Info ({ id, size }) {
  const [hovering, attrs] = useHover()
  return (
      <React.Fragment>
        {hovering === true
          ? <Tooltip id={id} />
          : null}
        <svg
          {...attrs}
          width={size}
          viewBox="0 0 16 16" >
            <path d="M9 8a1 1 0 0 0-1-1H5.5a1 1 0 1 0 0 2H7v4a1 1 0 0 0 2 0zM4 0h8a4 4 0       0 1 4 4v8a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4V4a4 4 0 0 1 4-4zm4 5.5a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3z" />
        </svg>
      </React.Fragment>
  )
}
```

There’s no convoluted logic flow you have to follow, you don’t have to create unnecessary wrapper components that modify your tree structure, and they compose with your app rather than against it.

To prove this even more, let’s look at how our code scales when we have to utilize multiple custom Hooks. As a reminder, I’ll also include the Higher-Order Component and Render Props solutions.

##### Higher-Order Component

```js
export default withHover(
  withTheme(
    withAuth(
      withRepos(Profile)
    )
  )
)
```

##### Render Props

```jsx
export default <Hover render={(hovering) => (
  <Theme render={(theme) => (
    <Auth render={(authed) => (
      <Repos render={(repos) => (
        <Profile 
          hovering={hovering}
          theme={theme}
          authed={authed}
          repos={repos}
        />
      )} />
    )} />
  )} />
)} />
```

##### Custom Hooks

```js
function Profile () {
  const [hovering, attrs] = useHover()
  const theme = useTheme()
  const [authed, toLogin] = useAuth()
  const repos = useRepos()

  ...
}
```