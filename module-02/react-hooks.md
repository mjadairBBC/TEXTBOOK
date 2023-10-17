# React Hooks

In 2018, the React team introduced Hooks to React. Hooks are essentially a way to 'hook in' to React *lifecycle* and *state* in a beautiful way. It doesn't add anything new to React functionality wise--its _syntactic sugar_ over classical components. However it hugely improves the readability and composability of your components.

### `State with hooks`

Here's how we would use a React hook for a counter app with `state`:

```js
import React, { useState } from 'react'
import ReactDOM from 'react-dom'

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      {/* Update my count in a pure functional way */}
      <button onClick={() => setCount(count + 1)}>
        Click me to count
      </button>
    </div>
  )
}

ReactDOM.render(
  <Counter />,
  document.getElementById('root')
)

```

`useState` is a hook for getting and setting a piece of state.

We initialise our counter state with an appropriate value (0 in this case, to start our count at 0). But for other examples, this might be an object, an array, or any other javascript type.

`useState` is a function that always returns an array of 2 elements. The line:
```
const [count, setCount] = useState(0);
```
_destructures_ that array into two named variables (we're choosing the names here):
- `count`: The value of the count itself
- `setCount`: A _function to update the count_ (this is going to replace our need to use *this.setState*)

Then in the returned JSX below:
```js
<p>You clicked {count} times</p>
{/* Update my count in a pure functional way */}
<button onClick={() => setCount(count + 1)}>
  Click me to count!
</button>
```
We use our `count` to display the count, and `setCount` when the button is clicked to increase the count by 1.

<br>

### `Lifecycle with hooks`

Hooks replace a lot of the need for using React lifecycle functions, like `componentWillMount`, directly. So far we've used these lifecycle functions for doing one-off API calls, so here's how we do that with the `useEffect` hook.

```js
import React, { useState, useEffect } from 'react'
import ReactDOM from 'react-dom'

const DonutsApp = () => {
  const [data, setData] = useState([]);
  useEffect(() => {
    fetch('https://ga-doughnuts.herokuapp.com/donuts')
      .then(resp => resp.json())
      .then(resp => setData(resp))
    return () => console.log('Unmounting component')
  }, [])

  return (
    <div>
      {data.map((item, id) => <p key={id}>{item.style}</p>)}
    </div>
  )
}

ReactDOM.render(
  <DonutsApp />,
  document.getElementById('root')
)

```

In order to capture the actual state of our donuts, we use `useState`, just like in the count example. We then store our API response as `data`, which we can then map over when we render our App.

> Note that `setData` and `data` are only responsible for representing donuts, so if we wanted to have multiple bits of state in our app, like counters, we would use `useState` multiple times for all the different pieces we need.

The main difference here though, is `useEffect`:

```js
useEffect(() => {
  fetch('https://ga-doughnuts.herokuapp.com/donuts')
    .then(resp => resp.json())
    .then(resp => setData(resp))
  return () => console.log('Unmounting component')
}, [])
```

This code will call the donuts API once when our component mounts, and set our `data` to be the donuts when our response comes back.

> If the component unmounts (which will never happen in this case), it'll log 'unmounting component'.

`useEffect` is a function that takes two arguments:
- A function to call.
- When to call this function.

For the second argument, there are 3 main cases. **This is important to understand**
- if this is an empty array `[]`, like in our example above, it will call the function once, when the component first renders (perfect for one-off API calls)
- If this array contains values `[a, b, c]`, where `a`, `b`, and `c` are pieces of state that can change, then this function will get called whenever any of these values change.
- If the second argument isn't given, the function will get called after *every render*. So not including an empty array as this second argument, in our example, would make infinite calls to our API, which we definitely don't want!

Optionally, like we've done above, you can *return* a function. This function will get called when the component unmounts. This can be useful when using React Router, if you want some action to happen when your page changes.

### Summary

That's it for the basics of hooks! Definitely check out the Dan Abramov video on hooks, which you can find below:

https://reactjs.org/docs/hooks-intro.html

And have a look at the other hooks that exist. These two we've learned above are by far the most common, and can replace our need to rely heavily on class-based React components!




