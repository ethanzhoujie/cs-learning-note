##### Why does React Hooks exist?

##### What problems does React Hooks solve?

#### React Evolution Path

##### React.createClass

at that time, JavaScript didn’t have a built-in class system.

```jsx
const ReposGrid = React.createClass({
  getInitialState () {
    return {
      repos: [],
      loading: true
    }
  },
  componentDidMount () {
    this.updateRepos(this.props.id)
  },
  componentDidUpdate (prevProps) {
    if (prevProps.id !== this.props.id) {
      this.updateRepos(this.props.id)
    }
  },
  updateRepos (id) {
    this.setState({ loading: true })

    fetchRepos(id)
      .then((repos) => this.setState({
        repos,
        loading: false
      }))
  },
  render() {
    const { loading, repos } = this.state

    if (loading === true) {
      return <Loading />
    }

    return (
      <ul>
        {repos.map(({ name, handle, stars, url }) => (
          <li key={name}>
            <ul>
              <li><a href={url}>{name}</a></li>
              <li>@{handle}</li>
              <li>{stars} stars</li>
            </ul>
          </li>
        ))}
      </ul>
    )
  }
})
```

##### React.Component

After ES6, react aligned react with the EcmaScript standard.

```jsx
class ReposGrid extends React.Component {
  constructor (props) {
    super(props)

    this.state = {
      repos: [],
      loading: true
    }

    this.updateRepos = this.updateRepos.bind(this)
  }
  componentDidMount () {
    this.updateRepos(this.props.id)
  }
  componentDidUpdate (prevProps) {
    if (prevProps.id !== this.props.id) {
      this.updateRepos(this.props.id)
    }
  }
  updateRepos (id) {
    this.setState({ loading: true })

    fetchRepos(id)
      .then((repos) => this.setState({
        repos,
        loading: false
      }))
  }
  render() {
    if (this.state.loading === true) {
      return <Loading />
    }

    return (
      <ul>
        {this.state.repos.map(({ name, handle, stars, url }) => (
          <li key={name}>
            <ul>
              <li><a href={url}>{name}</a></li>
              <li>@{handle}</li>
              <li>{stars} stars</li>
            </ul>
          </li>
        ))}
      </ul>
    )
  }
}
```

##### constructor

With Class components, you initialize the state of the component inside of the `constructor` method as a `state` property on the instance (`this`). However, according to the ECMAScript spec, if you’re extending a subclass (in this case, `React.Component`), you must first invoke `super` before you can use `this`. Specifically, when using React, you also have to remember to pass `props` to `super`.

```javascript
  constructor (props) {
    super(props) // 🤮

    ...
  }
```

##### autobinding

When using `createClass`, React would auto-magically bind all the methods to the component’s instance, `this`. With `React.Component`, that wasn’t the case. Very quickly, [React developers everywhere](https://stackoverflow.com/questions/32317154/react-uncaught-typeerror-cannot-read-property-setstate-of-undefined) realized they didn’t know how [the **this** keyword](https://tylermcginnis.com/this-keyword-call-apply-bind-javascript/) worked. Instead of having method invocations that “just worked”, you had to remember to `.bind` methods in the class’s `constructor`. If you didn’t, you’d get the popular “Cannot read property `setState` of undefined” error.

```jsx
  constructor (props) {
    ...

    this.updateRepos = this.updateRepos.bind(this) // 😭
  }
```

##### Class Fields

Class fields allow you to add instance properties directly as a property on a class without having to use `constructor`.

We no longer need to use `constructor` to set the initial state of the component and we no longer need to `.bind` in the `constructor` since we could use arrow functions for our methods.

```jsx
class ReposGrid extends React.Component {
  state = {
    repos: [],
    loading: true
  }
  componentDidMount () {
    this.updateRepos(this.props.id)
  }
  componentDidUpdate (prevProps) {
    if (prevProps.id !== this.props.id) {
      this.updateRepos(this.props.id)
    }
  }
  updateRepos = (id) => {
    this.setState({ loading: true })

    fetchRepos(id)
      .then((repos) => this.setState({
        repos,
        loading: false
      }))
  }
  render() {
    const { loading, repos } = this.state

    if (loading === true) {
      return <Loading />
    }

    return (
      <ul>
        {repos.map(({ name, handle, stars, url }) => (
          <li key={name}>
            <ul>
              <li><a href={url}>{name}</a></li>
              <li>@{handle}</li>
              <li>{stars} stars</li>
            </ul>
          </li>
        ))}
      </ul>
    )
  }
}
```

##### Duplicate Logic

##### Sharing Non-visual Logic

###### Higher-Order Component

```jsx
function withRepos (Component) {
  return class WithRepos extends React.Component {
    state = {
      repos: [],
      loading: true
    }
    componentDidMount () {
      this.updateRepos(this.props.id)
    }
    componentDidUpdate (prevProps) {
      if (prevProps.id !== this.props.id) {
        this.updateRepos(this.props.id)
      }
    }
    updateRepos = (id) => {
      this.setState({ loading: true })

      fetchRepos(id)
        .then((repos) => this.setState({
          repos,
          loading: false
        }))
    }
    render () {
      return (
        <Component
          {...this.props}
          {...this.state}
        />
      )
    }
  }
}
```

```jsx
// ReposGrid.js
function ReposGrid ({ loading, repos }) {
  ...
}

export default withRepos(ReposGrid)
```

```jsx
// Profile.js
function Profile ({ loading, repos }) {
  ...
}

export default withRepos(Profile)
```

Downside：

- Convoluted process

- Not easy to handle multi nested Higher-Order component

  ```jsx
  <WithHover>
    <WithTheme hovering={false}>
      <WithAuth hovering={false} theme='dark'>
        <WithRepos hovering={false} theme='dark' authed={true}>
          <Profile 
            id='JavaScript'
            loading={true} 
            repos={[]}
            authed={true}
            theme='dark'
            hovering={false}
          />
        </WithRepos>
      </WithAuth>
    <WithTheme>
  </WithHover>
  ```

###### Render Props



### Current State

So here’s where we’re at.

- React is hella popular.
- We use Classes for React components cause that’s what made the most sense at the time.
- Calling super(props) is annoying.
- No one knows how “this” works.
- OK, calm down. I know YOU know how “this” works, but it’s an unnecessary hurdle for some.
- Organizing our components by lifecycle methods forces us to sprinkle related logic throughout our components.
- React has no good primitive for sharing non-visual logic.

Now we need a new component API that solves all of those problems while remaining **simple**, **composable**, **flexible**, and **extendable**.



#### React Hooks

Sure we’d need to figure out a way to add the ability for functional components to have state and lifecycle methods, but assuming we did that, what benefits would we see?

Well, we would no longer have to call `super(props)`, we’d no longer need to worry about `bind`ing our methods or the `this` keyword, and we’d no longer have a use for `Class Fields`.

Issues need to be resolved:

##### State

`useState` takes in a single argument, the initial value for the state. What it returns is an array with the first item being the piece of state and the second item being a function to update that state.

As you can see, grabbing each item in the array individually isn’t the best developer experience. This is just to demonstrate how `useState` returns an array. Typically, you’d use Array Destructuring to grab the values in one line.

```jsx
const loadingTuple = React.useState(true)
const loading = loadingTuple[0]
const setLoading = loadingTuple[1]

...

loading // true
setLoading(false)
loading // false


// Or use array destructuring
// const loadingTuple = React.useState(true)
// const loading = loadingTuple[0]
// const setLoading = loadingTuple[1]

const [ loading, setLoading ] = React.useState(true) // 👌
```

##### Lifecycle methods

think in terms of **synchronization**

Defined, `useEffect` lets you perform side effects in function components. It takes two arguments, a function, and an optional array. The function defines which side effects to run and the (optional) array defines when to “re-sync” (or re-run) the effect.

```jsx
React.useEffect(() => {
  document.title = `Hello, ${username}`
}, [username])
```

Pretty slick, right? We’ve successfully gotten rid of `React.Component`, `constructor`, `super`, `this` and more importantly, we no longer have our effect logic sprinkled (and duplicated) throughout the component.

##### Sharing non-visual logic

There’s no built-in Hook for sharing non-visual logic, instead,you can create your own custom Hooks that are decoupled from any UI.

```jsx
function useRepos (id) {
  const [ repos, setRepos ] = React.useState([])
  const [ loading, setLoading ] = React.useState(true)

  React.useEffect(() => {
    setLoading(true)

    fetchRepos(id)
      .then((repos) => {
        setRepos(repos)
        setLoading(false)
      })
  }, [id])

  return [ loading, repos ]
}
```

```jsx
function ReposGrid ({ id }) {
  const [ loading, repos ] = useRepos(id)

  ...
}
```

```jsx
function Profile ({ user }) {
  const [ loading, repos ] = useRepos(user.id)

  ...
}
```





