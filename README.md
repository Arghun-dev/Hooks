# Hooks

### Basics

**Hooks are very powerful**
**Hooks allow you to write shorter stateful functional components**

`useState` & `useEffect` are the most common **hooks**

### Building Custom Hook (useToggle.js)

for example we have a function called `Toggler()` which we change a state from `true` to `false` and `fasle` to `true`.
So, we will create a `custom Hook` to add this logic to it. and always a custom name of custom Hooks always start with `use`.

`useToggle.js`

```js
import React, { useState } from 'react';

const useToggle(initialVal = false) {
  const [state, setState] = useState(initialVal);
  const toggle = () => setState(!state);
  
  // Return a piece of state AND a function to toggle it
  return [state, toggle];
}

export default useToggle;
```

### Using Custom Hook

And now we want to use the `useToggle` custom hook inside our component

```js
import React from 'react';
import useToggle from './hooks/useToggle.js';

const App = () => {
  const [isHappy, toggleIsHappy] = useToggle(true);
  const [isCorrect, toggleIsCorrect] = useToggle(false)
  
  return (
    <>
      <h1 onClick={toggleIsHappy}>{isHappy ? 'I am happy' : 'I am angry'}</h1>
      <h1 onClick={toggleIsCorrect}>{isCorrect ? 'correct' : 'wrong'}</h1>
    </>
    
  )
}
```

### useEffect

**You can think of useEffect Hook as `componentDidMount`, `componentDidUpdate` and `componentWillUnmount` combined**

#### What does useEffect do? 
By using this Hook, you tell React that your component needs to do something after render. React will remember the function you passed (we’ll refer to it as our “effect”), and call it later after performing the DOM updates. In this effect, we set the document title, but we could also perform data fetching or call some other imperative API.

#### Does useEffect run after every render?
Yes! By default, it runs both after the first render and after every update. (We will later talk about how to customize this.) Instead of thinking in terms of “mounting” and “updating”, you might find it easier to think that effects happen “after render”. React guarantees the DOM has been updated by the time it runs the effects.

#### Detailed Explanation

```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```

We declare the count state variable, and then we tell React we need to use an effect. We pass a function to the useEffect Hook. This function we pass is our effect. Inside our effect, we set the document title using the document.title browser API. We can read the latest count inside the effect because it’s in the scope of our function. When React renders our component, it will remember the effect we used, and then run our effect after updating the DOM. This happens for every render, including the first one.

Experienced JavaScript developers might notice that the function passed to useEffect is going to be different on every render. This is intentional. In fact, this is what lets us read the count value from inside the effect without worrying about it getting stale. Every time we re-render, we schedule a different effect, replacing the previous one. In a way, this makes the effects behave more like a part of the render result — each effect “belongs” to a particular render. We will see more clearly why this is useful later on this page.

`Tip:

Unlike componentDidMount or componentDidUpdate, effects scheduled with useEffect don’t block the browser from updating the screen. This makes your app feel more responsive. The majority of effects don’t need to happen synchronously. In the uncommon cases where they do (such as measuring the layout), there is a separate useLayoutEffect Hook with an API identical to useEffect.
`

#### Effects with Cleanup

Earlier, we looked at how to express side effects that don’t require any cleanup. However, some effects do. For example, we might want to set up a subscription to some external data source. In that case, it is important to clean up so that we don’t introduce a memory leak! Let’s compare how we can do it with classes and with Hooks.


##### Example Using Classes

In a React class, you would typically set up a subscription in `componentDidMount`, and clean it up in `componentWillUnmount`. For example, let’s say we have a ChatAPI module that lets us subscribe to a friend’s online status. Here’s how we might subscribe and display that status using a class:

```js
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

Notice how `componentDidMount` and `componentWillUnmount` need to mirror each other. Lifecycle methods force us to split this logic even though conceptually code in both of them is related to the same effect.


##### Example Using Hooks

You might be thinking that we’d need a separate effect to perform the cleanup. But code for adding and removing a subscription is so tightly related that useEffect is designed to keep it together. If your effect returns a function, React will run it when it is time to clean up:

```js
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

#### Why did we return a function from our effect?
This is the optional cleanup mechanism for effects. Every effect may return a function that cleans up after it. This lets us keep the logic for adding and removing subscriptions close to each other. They’re part of the same effect!

#### When exactly does React clean up an effect? 
React performs the cleanup when the component unmounts. However, as we learned earlier, effects run for every render and not just once. This is why React also cleans up effects from the previous render before running the effects next time. We’ll discuss why this helps avoid bugs and how to opt out of this behavior in case it creates performance issues later below.

**We don’t have to return a named function from the effect. We called it cleanup here to clarify its purpose, but you could return an arrow function or call it something different.**


### Recap

We’ve learned that useEffect lets us express different kinds of side effects after a component renders. Some effects might require cleanup so they return a function:

```js
useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

Other effects might not have a cleanup phase, and don’t return anything.

```js
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

#### Tip: Use Multiple Effects to Separate Concerns

One of the problems we outlined in the Motivation for Hooks is that class lifecycle methods often contain unrelated logic, but related logic gets broken up into several methods. Here is a component that combines the counter and the friend status indicator logic from the previous examples:

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

Note how the logic that sets document.title is split between componentDidMount and componentDidUpdate. The subscription logic is also spread between componentDidMount and componentWillUnmount. And componentDidMount contains code for both tasks.

So, how can Hooks solve this problem? Just like you can use the State Hook more than once, you can also use several effects. This lets us separate unrelated logic into different effects:

```js
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

Hooks let us split the code based on what it is doing rather than a lifecycle method name. React will apply every effect used by the component, in the order they were specified.


#### Explanation: Why Effects Run on Each Update

If you’re used to classes, you might be wondering why the effect cleanup phase happens after every re-render, and not just once during unmounting. Let’s look at a practical example to see why this design helps us create components with fewer bugs.

Earlier on this page, we introduced an example FriendStatus component that displays whether a friend is online or not. Our class reads friend.id from this.props, subscribes to the friend status after the component mounts, and unsubscribes during unmounting:

```js
componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
 ```
 
 **But what happens if the friend prop changes**
 while the component is on the screen? Our component would continue displaying the online status of a different friend. This is a bug. We would also cause a memory leak or crash when unmounting since the unsubscribe call would use the wrong friend ID.
 
 In a class component, we would need to add `componentDidUpdate` to handle this case.
 
 ```js
 componentDidUpdate(prevProps) {
    // Unsubscribe from the previous friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Subscribe to the next friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
 ```
 
Forgetting to handle componentDidUpdate properly is a common source of bugs in React applications.

Now consider the version of this component that uses Hooks:

```js
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

It doesn’t suffer from this bug. (But we also didn’t make any changes to it.)

There is no special code for handling updates because useEffect handles them by default. It cleans up the previous effects before applying the next effects. To illustrate this, here is a sequence of subscribe and unsubscribe calls that this component could produce over time:

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

This behavior ensures consistency by default and prevents bugs that are common in class components due to missing update logic.


#### Tip: Optimizing Performance by Skipping Effects
In some cases, cleaning up or applying the effect after every render might create a performance problem. In class components, we can solve this by writing an extra comparison with prevProps or prevState inside componentDidUpdate:

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

This requirement is common enough that it is built into the useEffect Hook API. You can tell React to skip applying an effect if certain values haven’t changed between re-renders. To do so, pass an `array` as an optional `second argument` to `useEffect`:

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

In the example above, we pass [count] as the second argument. What does this mean? If the count is 5, and then our component re-renders with count still equal to 5, React will compare [5] from the previous render and [5] from the next render. Because all items in the array are the same (5 === 5), React would skip the effect. That’s our optimization.

When we render with count updated to 6, React will compare the items in the [5] array from the previous render to items in the [6] array from the next render. This time, React will re-apply the effect because 5 !== 6. If there are multiple items in the array, React will re-run the effect even if just one of them is different.

This also works for effects that have a cleanup phase:

```js
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

**If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array `([])` as a second argument. **
This tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run. This isn’t handled as a special case — it follows directly from how the dependencies array always works.


# Rules of Hooks

## Only Call Hooks at the Top Level
**Don’t call Hooks inside loops, conditions, or nested functions**
Instead, always use Hooks at the top level of your React function. By following this rule, you ensure that Hooks are called in the same order each time a component renders. That’s what allows React to correctly preserve the state of Hooks between multiple useState and useEffect calls. (If you’re curious, we’ll explain this in depth below.)

## Only Call Hooks from React Functions
**Don’t call Hooks from regular JavaScript functions. Instead, you can:**

✅ Call Hooks from React function components.
✅ Call Hooks from custom Hooks (we’ll learn about them on the next page).
By following this rule, you ensure that all stateful logic in a component is clearly visible from its source code.

# ESLint Plugin

We released an ESLint plugin called `eslint-plugin-react-hooks` that enforces these two rules. You can add this plugin to your project if you’d like to try it:

This plugin is included by default in Create React App.

`$. npm install eslint-plugin-react-hooks --save-dev`

```js
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

# Building Your Own Hooks

Building your own Hooks lets you extract component logic into reusable functions.
When we were learning about using the Effect Hook, we saw this component from a chat application that displays a message indicating whether a friend is online or offline:

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

Now let’s say that our chat application also has a contact list, and we want to render names of online users with a green color. We could copy and paste similar logic above into our FriendListItem component but it wouldn’t be ideal:

```js
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

Instead, we’d like to share this logic between FriendStatus and FriendListItem.

Traditionally in React, we’ve had two popular ways to share stateful logic between components: `render props` and `higher-order components`. We will now look at how Hooks solve many of the same problems without forcing you to add more components to the tree.

## Extracting a Custom Hook
When we want to share logic between two JavaScript functions, we extract it to a third function. Both components and Hooks are functions, so this works for them too!

A custom Hook is a JavaScript function whose name starts with `use` and that may call other Hooks. For example,`useFriendStatus` below is our first custom Hook:

```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

## Using a Custom Hook

In the beginning, our stated goal was to remove the duplicated logic from the FriendStatus and FriendListItem components. Both of them want to know whether a friend is online.

Now that we’ve extracted this logic to a `useFriendStatus hook`, we can just use it:

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

### Do two components using the same Hook share state?
No. Custom Hooks are a mechanism to reuse stateful logic (such as setting up a subscription and remembering the current value), but every time you use a custom Hook, all state and effects inside of it are fully isolated.

### How does a custom Hook get isolated state?
Each call to a Hook gets isolated state. Because we call useFriendStatus directly, from React’s point of view our component just calls useState and useEffect. And as we learned earlier, we can call useState and useEffect many times in one component, and they will be completely independent.


## Tip: Pass Information Between Hooks
Since Hooks are functions, we can pass information between them.

To illustrate this, we’ll use another component from our hypothetical chat example. This is a chat message recipient picker that displays whether the currently selected friend is online:

```js
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```

We keep the currently chosen `friend ID` in the `recipientID` state variable, and update it if the user chooses a different friend in the <select> picker.

Because the useState Hook call gives us the latest value of the recipientID state variable, we can pass it to our custom useFriendStatus Hook as an argument:

```js
const [recipientID, setRecipientID] = useState(1);
const isRecipientOnline = useFriendStatus(recipientID);
```

This lets us know whether the currently selected friend is online. If we pick a different friend and update the recipientID state variable, our useFriendStatus Hook will unsubscribe from the previously selected friend, and subscribe to the status of the newly selected one.


### useYourImagination()

Custom Hooks offer the flexibility of sharing logic that wasn’t possible in React components before. You can write custom Hooks that cover a wide range of use cases like form handling, animation, declarative subscriptions, timers, and probably many more we haven’t considered. What’s more, you can build Hooks that are just as easy to use as React’s built-in features.

Try to resist adding abstraction too early. Now that function components can do more, it’s likely that the average function component in your codebase will become longer. This is normal — don’t feel like you have to immediately split it into Hooks. But we also encourage you to start spotting cases where a custom Hook could hide complex logic behind a simple interface, or help untangle a messy component.

For example, maybe you have a complex component that contains a lot of local state that is managed in an ad-hoc way. useState doesn’t make centralizing the update logic any easier so you might prefer to write it as a `Redux reducer`:


```js
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, {
        text: action.text,
        completed: false
      }];
    // ... other actions ...
    default:
      return state;
  }
}
```

Reducers are very convenient to test in isolation, and scale to express complex update logic. You can further break them apart into smaller reducers if necessary. However, you might also enjoy the benefits of using React local state, or might not want to install another library.

So what if we could write a `useReducer Hook` that lets us manage the local state of our component with a reducer? A simplified version of it might look like this:

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

Now we could use it in our component, and let the reducer drive its state management:

```js
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer, []);

  function handleAddClick(text) {
    dispatch({ type: 'add', text });
  }

  // ...
}
```

The need to manage local state with a reducer in a complex component is common enough that we’ve built the useReducer Hook right into React. You’ll find it together with other built-in Hooks in the Hooks API reference.


# Hooks API Reference

## Basic Hooks

```js
const [state, setState] = useState(initialState);
```

Returns a stateful value, and a function to update it.

During the initial render, the returned state (state) is the same as the value passed as the first argument (initialState).

The setState function is used to update the state. It accepts a new state value and enqueues a re-render of the component.

```js
setState(newState);
```

**Functional updates**
If the new state is computed using the previous state, you can pass a function to setState. The function will receive the previous value, and return an updated value. Here’s an example of a counter component that uses both forms of setState:

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

The ”+” and ”-” buttons use the functional form, because the updated value is based on the previous value. But the “Reset” button uses the normal form, because it always sets the count back to the initial value.

If your update function returns the exact same value as the current state, the subsequent rerender will be skipped completely.

Note

Unlike the setState method found in class components, useState does not automatically merge update objects. You can replicate this behavior by combining the function updater form with object spread syntax:

```js
setState(prevState => {
  // Object.assign would also work
  return {...prevState, ...updatedValues};
});
```

Another option is `useReducer`, which is more suited for managing state objects that contain multiple sub-values.

## useEffect

Accepts a function that contains imperative, possibly effectful code.

Mutations, subscriptions, timers, logging, and other side effects are not allowed inside the main body of a function component (referred to as React’s render phase). Doing so will lead to confusing bugs and inconsistencies in the UI.

Instead, use useEffect. The function passed to useEffect will run after the render is committed to the screen. Think of effects as an escape hatch from React’s purely functional world into the imperative world.

By default, effects run after every completed render, but you can choose to fire them only when certain values have changed.

### Cleaning up an effect

Often, effects create resources that need to be cleaned up before the component leaves the screen, such as a subscription or timer ID. To do this, the function passed to useEffect may return a clean-up function. For example, to create a subscription:

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

The clean-up function runs before the component is removed from the UI to prevent memory leaks. Additionally, if a component renders multiple times (as they typically do), the previous effect is cleaned up before executing the next effect. In our example, this means a new subscription is created on every update. To avoid firing an effect on every update, refer to the next section.

### Conditionally firing an effect

The default behavior for effects is to fire the effect after every completed render. That way an effect is always recreated if one of its dependencies changes.

However, this may be overkill in some cases, like the subscription example from the previous section. We don’t need to create a new subscription on every update, only if the source prop has changed.

To implement this, pass a second argument to useEffect that is the array of values that the effect depends on. Our updated example now looks like this:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

Now the subscription will only be recreated when `props.source` changes.

## useContext

```js
const value = useContext(MyContext);
```

Don’t forget that the argument to useContext must be the context object itself:

Correct: useContext(MyContext)
Incorrect: useContext(MyContext.Consumer)
Incorrect: useContext(MyContext.Provider)

A component calling useContext will always re-render when the context value changes. If re-rendering the component is expensive, you can optimize it by using `memoization`.


# Additional Hooks

The following Hooks are either variants of the basic ones from the previous section, or only needed for specific edge cases. Don’t stress about learning them up front.

## useReducer

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

An alternative to `useState`. Accepts a reducer of type `(state, action) => newState`, and returns the current state paired with a dispatch method. (If you’re familiar with Redux, you already know how this works.)

`useReducer` is usually preferable to `useState` when you have `complex state logic` that involves `multiple sub-values` or when the `next state depends on the previous one`. `useReducer` also lets you optimize performance for components that trigger deep updates because you can pass dispatch down instead of callbacks.

Here’s the counter example from the useState section, rewritten to use a reducer:

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

**Specifying the initial state**
There are two different ways to initialize `useReducer state`. You may choose either one depending on the use case. The simplest way is to pass the initial state as a second argument:

```js
const [state, dispatch] = useReducer(
    reducer,
    {count: initialCount}
);
```


# useEffect examples

`useEffect` runs every signle render including the first rendering.

```js
import React, { useState, useEffect } from "react";

const Clicker = () => {
  const [count, setCount] = useState(0);
  useEffect(() => {
    alert("something chnaged");
  }, [count]);
  return <button onClick={() => setCount(count + 1)}>Clicked! {count}</button>;
};

export default Clicker;
```

# Fetching Data w the useEffect Hook

in here we have a select which we make an API call everytime we select a number. And we want to make that call every time that number changes.

```js
import React, { useState, useEffect } from "react";
import axios from "axios";

const SwMovies = () => {
  const [movieNumber, setMovieNumber] = useState(1);
  const [movie, setMovie] = useState("");

  useEffect(() => {
    async function getData() {
      const response = await axios.get(
        `https://swapi.dev/api/people/${movieNumber}/`
      );
      setMovie(response.data);
    }
    getData();
  }, [movieNumber]);

  return (
    <>
      <h4>{movie.name}</h4>
      <select
        value={movieNumber}
        onChange={(e) => setMovieNumber(e.target.value)}
      >
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
        <option value="7">7</option>
      </select>
      <div>
        <p></p>
      </div>
    </>
  );
};

export default SwMovies;
```

And Now If we had another state, which we want to make API call whenever that state changes. means we have 2 states which we want to make api call whenever these states chnages.
for example we want to make Ajax call whenever the `number` or `year` change.

```js
useEffect(() => {
    async function getData() {
      const response = await axios.get(
        `https://swapi.dev/api/people/${movieNumber}/`
      );
      setMovie(response.data);
    }
    getData();
  }, [number, year]);
```

or you can write theme in separate `useEffects`
