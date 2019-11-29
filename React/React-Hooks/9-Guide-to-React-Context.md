## React Context

> Context provides a way to pass data through the component tree without having to pass props down manually at every level. - The React Docs

### The Context API

```jsx
const LocaleContext = React.createContext()
```

Now if we examine our `LocaleContext`, you’ll notice that it has two properties, both of which are React components, `Provider`, and `Consumer`.

`Provider` allows us to “declare the data that we want available throughout our component tree”.

`Consumer` allows “any component in the component tree that needs that data to be able to subscribe to it”.

### Provider

You use `Provider` just like you would any other React component. It accepts a `value` prop which is the data that you want available to any of its `children` who need to consume it.

```jsx
<MyContext.Provider value={data}>
  <App />
</MyContext.Provider>
```

In our example, we want `locale` to be available anywhere in the component tree. We also want to update the UI (re-render) whenever it changes, so we’ll stick it on our component’s state.

```jsx
// LocaleContext.js
import React from "react"

const LocaleContext = React.createContext()

export default LocaleContext
```

```jsx
import React from 'react'
import LocaleContext from './LocaleContext'

export default function App () {
  const [locale, setLocale] = React.useState('en')

  return (
    <LocaleContext.Provider value={locale}>
      <Home />
    </LocaleContext.Provider>
  )
}
```

Now, any component in our component tree that needs the value of `locale` will have the option to subscribe to it using `LocaleContext.Consumer`.

### Consumer

Again, the whole point of the `Consumer` component is it allows you to get access to the data that was passed as a `value` prop to the Context’s `Provider` component. To do this, `Consumer` uses a [render prop](https://tylermcginnis.com/react-render-props).

```jsx
<MyContext.Consumer>
  {(data) => {
    return (
      <h1>
        The "value" prop passed to "Provider" was {data}
      </h1>
    )
  }}
</MyContext.Consumer>
```

Now in our example, because we passed `locale` as the `value` prop to `LocaleContext.Provider`, we can get access to it by passing `LocaleContext.Consumer` a render prop.

```jsx
// Blog.js
import React from 'react'
import LocaleContext from './LocaleContext'

export default function Blog () {
  return (
    <LocaleContext.Consumer>
      {(locale) => <Posts locale={locale} />}
    </LocaleContext.Consumer>
  )
}
```

### Updating Context State

At this point, we’ve seen that because we wrapped our whole app in `<LocaleContext.Provider value={locale}>` , any component in our application tree can get access to `locale` by using `LocaleContext.Consumer`. However, what if we also want to be able to toggle it (`en` -> `es`) from anywhere inside of our component tree?

Your first intuition might be to do something like this.

```jsx
export default function App () {
  const [locale, setLocale] = React.useState('en')

  const toggleLocale = () => {
    setLocale((locale) => {
      return locale === 'en' ? 'es' : 'en'
    })
  }

  return (
    <LocaleContext.Provider value={{
      locale,
      toggleLocale
    }}>
      <Home />
    </LocaleContext.Provider>
  )
}
```

What we’ve done is added a new property to the object we pass to `value`. Now, anywhere in our component tree, using `ThemeContext.Consumer`, we can grab `locale` OR `toggleLocale`.

Sadly, the idea is right, but the execution is a little off. Can you think of any downsides to this approach? Hint, it has to do with performance.

Just like React re-renders with prop changes, whenever the data passed to `value` changes, React will re-render every component which used `Consumer` to subscribe to that data. The way in which React knows if the data changes is by using “reference identity” (which is kind of a fancy way of saving `oldObject` === `newObject`).

Currently with how we have it set up (`value={{}}`), we’re passing a **new** object to `value` every time that `App`re-renders. What this means is that when React checks if the data passed to `value` has changed, it’ll always think it has since we’re always passing in a new object. As a result of that, every component which used `Consumer` to subscribe to that data will re-render as well, even if `locale` or `toggleLocale` didn’t change.

To fix this, instead of passing a **new** object to `value` every time, we want to give it a reference to an object it already knows about. To do this, we can use the `useMemo` Hook.

```jsx
export default function App () {
  const [locale, setLocale] = React.useState('en')

  const toggleLocale = () => {
    setLocale((locale) => {
      return locale === 'en' ? 'es' : 'en'
    })
  }

  const value = React.useMemo(() => ({
    locale,
    toggleLocale
  }), [locale])

  return (
    <LocaleContext.Provider value={value}>
      <Home />
    </LocaleContext.Provider>
  )
}
```

React will make sure the `value` that `useMemo` returns stays the same unless `locale` changes. This way, any component which used `Consumer` to subscribe to our `locale` context will only re-render if `locale` changes.

Now, anywhere inside of our component tree, we can get access to the `locale` value or the ability to change it via `toggleLocale`.

### defaultValue

Whenever you render a `Consumer` component, it gets its value from the `value` prop of the nearest `Provider`component of the same Context object. However, what if there isn’t a parent `Provider` of the same Context object? In that case, it’ll get its value from the first argument that was passed to `createContext` when the Context object was created.

```jsx
const MyContext = React.creatContext('defaultValue')
```

And adapted to our example.

```jsx
const LocaleContext = React.createContext('en')
```

Now, if we use `` without previously rendering a ``, the value passed to `Consumer` will be `en`.

### useContext

At this point, you’ve seen that in order to get access to the data that was passed as a `value` prop to the Context’s `Provider` component, you use `Consumer` as a render prop.

```jsx
export default function Nav () {
  return (
    <LocaleContext.Consumer>
      {({ locale, toggleLocale }) => locale === "en" 
        ? <EnglishNav toggleLocale={toggleLocale} />
        : <SpanishNav toggleLocale={toggleLocale} />}
    </LocaleContext.Consumer>
  );
}
```

`useContext` takes in a Context object as its first argument and returns whatever was passed to the `value` prop of the nearest `Provider` component. Said differently, it has the same use case as `.Consumer` but with a more composable API.

```jsx
export default function Nav () {
  const { locale, toggleLocale } = React.useContext(
    LocaleContext
  )

  return locale === 'en'
    ? <EnglishNav toggleLocale={toggleLocale} />
    : <SpanishNav toggleLocale={toggleLocale} />
}
```

This works, but as always the render-props syntax is a little funky. The problem gets worse if you have multiple context values you need to grab.

```jsx
export default function Nav () {
  return (
    <AuthedContext.Consumer>
      {({ authed }) => authed === false
        ? <Redirect to='/login' />
        : <LocaleContext.Consumer>
            {({ locale, toggleLocale }) => locale === "en" 
              ? <EnglishNav toggleLocale={toggleLocale} />
              : <SpanishNav toggleLocale={toggleLocale} />}
          </LocaleContext.Consumer>}
    </AuthedContext.Consumer>
  ) 
}
```

Oof. Luckily for us, there’s a Hook that solves this problem - `useContext`. `useContext` takes in a Context object as its first argument and returns whatever was passed to the `value` prop of the nearest `Provider` component. Said differently, it has the same use case as `.Consumer` but with a more composable API.

As always, this API really shines when you need to grab multiple values from different Contexts.

```jsx
export default function Nav () {
  const { authed } = React.useContext(AuthedContext)

  if (authed === false) {
    return <Redirect to='/login' />
  }

  const { locale, toggleLocale } = React.useContext(
    LocaleContext
  )

  return locale === 'en'
    ? <EnglishNav toggleLocale={toggleLocale} />
    : <SpanishNav toggleLocale={toggleLocale} />
}
```

