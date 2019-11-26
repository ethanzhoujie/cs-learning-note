## The useRef Hook

The `useState` Hook gives you two things - a value that will persist across renders and an API to update that value and trigger a re-render.

```jsx
const [value, setValueAndReRender] = React.useState(
  'initial value'
)
```

`useRef` the ability to persist a value across renders without causing a re-render

```jsx
function Counter() {
  const [count, setCount] = React.useState(0)
  const id = React.useRef(null)

  const clearInterval = () => {
    window.clearInterval(id.current)
  }

  React.useEffect(() => {
    id.current = window.setInterval(() => {
      setCount(c => c + 1)
    }, 1000)

    return clearInterval
  }, [])

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={clearInterval}>Stop</button>
    </div>
  )
}
```

`useRef` follows the same API we created earlier. It accepts an initial value as its first argument and it returns an object that has a `current` property (which will initially be set to whatever the initial value was). From there, anything you add to `current` will be persisted across renders.

The most popular use case for `useRef` is getting access to DOM nodes. If you pass the value you get from `useRef` as a `ref` prop on any React element, React will set the `current` property to the corresponding DOM. This allows you to do things like grab input values or set focus.

```jsx
function Form () {
  const nameRef = React.useRef()
  const emailRef = React.useRef()
  const passwordRef = React.useRef()

  const handleSubmit = e => {
    e.preventDefault()

    const name = nameRef.current.value
    const email = emailRef.current.value
    const password = passwordRef.current.value

    console.log(name, email, password)
  }

  return (
    <React.Fragment>
      <label>
        Name:
        <input 
          placeholder="name"
          type="text"
          ref={nameRef} 
        />
      </label>
      <label>
        Email:
        <input 
          placeholder="email"
          type="text"
          ref={emailRef}
                  />
      </label>
      <label>
        Password:
        <input 
          placeholder="password"
          type="text"
          ref={passwordRef} 
        />
      </label>

      <hr />

      <button onClick={() => nameRef.current.focus()}>
        Focus Name Input
      </button>
      <button onClick={() => emailRef.current.focus()}>
        Focus Email Input
      </button>
      <button onClick={() => passwordRef.current.focus()}>
        Focus Password Input
      </button>

      <hr />

      <button onClick={handleSubmit}>Submit</button>
    </React.Fragment>
  )
}
```

