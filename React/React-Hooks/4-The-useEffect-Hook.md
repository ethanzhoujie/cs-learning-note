

### useEffect

Mutating non-local variables, making network requests, and updating the DOM are all examples of common side effects.

`useEffect` allows you to perform side effects in function components.

#### Add an effect

```jsx
React.useEffect(() => {
  document.title = 'The new title.'
})
```

By default, React will re-invoke the effect **after every render**.

by default, on every render (including the initial render), the effect won’t run until **after** React has updated the DOM and the browser has painted those updates to the view. The reason for this timing is so the side effect doesn’t block updates to the UI.

#### Skipping Side Effect

Given an array of values to `useEffect`, React can infer when to re-invoke the effect based on if any of those values change between renders.

```javascript
React.useEffect(() => {
  // Will be invoked on the initial render 
  // and all subsequent re-renders.
})
```

```javascript
React.useEffect(() => {
  // Will be invoked on the initial render
  // and when "id" or "authed" changes.
}, [id, authed])
```

```javascript
React.useEffect(() => {
  // Will only be invoked on the initial render
}, [])
```

#### Cleaning up side effects

```javascript
React.useEffect(() => {

  return () => {
    // invoked right before invoking
    // the new effect on a re-render AND
    // right before removing the component
    // from the DOM
  }
})
```

```javascript
function Profile ({ username }) {
  const [profile, setProfile] = React.useState(null)

  React.useEffect(() => {
    subscribe(username, setProfile)

    return () => {
      unsubscribe(username)
      setProfile(null)
    }
  }, [username])

  if (profile === null) {
    return <p>Loading...</p>
  }

  return (
    <React.Fragment>
      <h1>@{profile.login}</h1>
      <img
        src={profile.avatar_url}
        alt={`Avatar for ${profile.login}`}
      />
      <p>{profile.bio}</p>
    </React.Fragment>
  );
}
```

In this example, there are two scenarios where the cleanup function would be invoked. First, whenever `username` changes, before the new effect is invoked with the new `username`. And second, right before `Profile` is removed from the DOM. In both scenarios, we want to `unsubscribe` from our API as well as reset `profile` to `null` so we get the `Loading...` UI.

So we can better see this process, let’s assume our `Profile` component is rendered initially with the `username``tylermcginnis`, then a user event triggers a re-render with the `username` `sdras`.

```markdown
Initial Render
  username prop: 'tylermcginnis'
  profile: null
  Effect:
    () => subscribe('tylermcginnis', setProfile)
  Cleanup:
    () => unsubscribe('tylermcginnis')
  Description of UI: Loading...

  React: Updates the DOM
  Browser: Re-paints with DOM updates
  React invokes Effect: 
    () => subscribe('tylermcginnis', setProfile)


setProfile was invoked
  React updates 'profile', causing a re-render


Next Render
  username prop: 'tylermcginnis'
  profile: {login: 'tylermcginnis', name: 'Tyler McGinnis'}...
  Effect:
    () => subscribe('tylermcginnis', setProfile)
  Cleanup:
    () => unsubscribe('tylermcginnis')
  Description of UI: <h1>@tylermcginnis</h1>...

  React: Updates the DOM
  Browser: Re-paints with DOM updates
  Effect: Skipped since 'username' didn't change.


User event changes 'username' to 'sdras'


Next Render
  username prop: 'sdras'
  profile: {login: 'tylermcginnis', name: 'Tyler McGinnis'}...
  Effect:
    () => subscribe('sdras', setProfile)
  Cleanup:
    () => unsubscribe('sdras')
  Description of UI: <h1>@tylermcginnis</h1>...

  React: Updates the DOM
  Browser: Re-paints with DOM updates
  React invokes previous render's cleanup:
    () => unsubscribe('tylermcginnis')
  React invokes Effect: 
    () => subscribe('sdras', setProfile)


setProfile was invoked
  React updates "profile", causing a re-render


Next Render
  username prop: 'sdrsa'
  profile: {login: 'sdrsa', name: 'Sarah Drasner'}...
  Effect:
    () => subscribe('sdrsa', setProfile)
  Cleanup:
    () => unsubscribe('sdras')
  Description of UI: <h1>@sdrsa</h1>...

  React: Updates the DOM
  Browser: Re-paints with DOM updates
  Effect: Skipped since 'username' didn't change.
```

#### useEffect vs Lifecycle Events

You may have noticed that up until this point, I’ve been deliberate about not making comparisons between `useEffect` and the traditional component lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). There are a few reasons for that. First, it creates an unnecessary pre-requisite for someone new to React. As we’ve seen, you don’t need to know anything about the traditional lifecycle methods to understand `useEffect`. Second, they’re two fundamentally different mental models. By comparing them upfront you send a signal, often unintentionally, that `useEffect` is just a new way of adding lifecycle events to our function components. As you’ve seen by now, this couldn’t be further from the truth.