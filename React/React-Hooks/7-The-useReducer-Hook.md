### The useReducer Hook

#### Reduce

Reduce is a functional programming pattern that takes a collection (an array or object) as input and returns a single value as output.

The key difference between `reduce` and `forEach` is that `reduce` is able to keep track of the accumulated state internally without relying upon or modifying state outside of its own scope - that’s what makes it a *pure* function. The way it does this is, for each element in the collection, it invokes a reducer function passing it two arguments, the accumulated state and the current element in the collection. What the reducer function returns will be passed as the first argument to the next invocation of the reducer and will eventually result in the final value.

```javascript
const nums = [2,4,6]
const initialState = 0

function reducer (state, value) {
  return state + value
}

const total = nums.reduce(reducer, initialState)
```

#### useReducer

`useReducer` that allows you to add state to a function component but manage that state using the reducer pattern.

Instead of just returning the state, as we mentioned earlier, we need a way for user actions to invoke our reducer function. Because of this, `useReducer` returns an array with the first element being the `state` and the second element being a `dispatch` function which when called, will invoke the `reducer`.

```js
const [state, dispatch] = React.useReducer(
  reducer, 
  initialState
)
```

When invoked, whatever you pass to `dispatch` will be passed as the second argument to the `reducer` (which we’ve been calling `value`). The first argument (which we’ve been calling `state`) will be passed implicitly by React and will be whatever the previous `state` value was. Putting it all together, here’s our code.

```javascript
function reducer (state, value) {
  return state + value
}

function Counter () {
  const [count, dispatch] = React.useReducer(
    reducer, 
    0
  )

  return (
    <React.Fragment>
      <h1>{count}</h1>
      <button onClick={() => dispatch(1)}>
        +
      </button>
    </React.Fragment>
  );
}
```

The flow is the exact same as our diagram above. Whenever the `+` button is clicked, `dispatch` will be invoked. That will call `reducer` passing it two arguments, `state`, which will come implicitly from React, and `value`, which will be whatever was passed to `dispatch`. What we return from `reducer` will become our new `count`. Finally, because `count` changed, React will re-render the component, updating the UI.

```javascript
function reducer (state, action) {
  if (action === 'increment') {
    return state + 1
  } else if (action === 'decrement') {
    return state - 1
  } else if (action === 'reset') {
    return 0
  } else {
    throw new Error(`This action type isn't supported.`)
  }
}

function Counter() {
  const [count, dispatch] = React.useReducer(
    reducer,
    0
  )

  return (
    <React.Fragment>
      <h1>{count}</h1>
      <button onClick={() => dispatch('increment')}>
        +
      </button>
      <button onClick={() => dispatch('decrement')}>
        -
      </button>
      <button onClick={() => dispatch('reset')}>
        Reset
      </button>
    </React.Fragment>
  );
}
```

### useState vs useReducer

Fundamentally, `useState` and `useReducer` accomplish the same thing - they both allow us to add state to function components.

#### Declarative state updates

The reason `useReducer` can be more declarative is because it allows us to map actions to state transitions. This means, instead of having a collection of `setX` invocations, we can simply `dispatch`the action type that occurred. Then our `reducer` can encapsulate the imperative, instructional code.

#### Update state based on another piece of state

Whenever updating one piece of state depends on the value of another piece of state, reach for `useReducer`.

#### Minimize Dependency Array

Part of mastering the `useEffect` Hook is learning how to properly manage its second argument, the dependency array.

```js
React.useEffect(() => {
  // side effect
}, [/* dependency array */])
```

Leave it off and you could run into an infinite loop scenario. Forget to add values your effect depends on and you’ll have stale data. Add too many values and your effect won’t be re-invoked when it needs to be.

It may come as a surprise, but `useReducer` is one strategy for improving the management of the dependency array. The reason for this goes back to what we’ve mentioned a few times now, `useReducer` allows you to decouple how the state is updated from the action that triggered the update. In practical terms, because of this decoupling, you can exclude values from the dependency array since the effect only `dispatch`es the type of action that occurred and doesn’t rely on any of the state values (which are encapsulated inside of the `reducer`). That was a lot of words, here’s some code.

```js
React.useEffect(() => {
  setCount(count + 1)
}, [count])

React.useEffect(() => {
  dispatch({ 
    type: 'increment'
  })
}, [])
```

### Summary

`useState` and `useReducer` both allow you to add state to function components. `useReducer` offers a bit more flexibility since it allows you to decouple how the state is updated from the action that triggered the update - typically leading to more declarative state updates.

If different pieces of state update independently from one another (`hovering`, `selected`, etc.), `useState` should work fine. If your state tends to be updated together or if updating one piece of state is based on another piece of state, go with `useReducer`.