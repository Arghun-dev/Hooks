# Hooks

### Basics

**Hooks are very powerful**
**Hooks allow you to write shorter stateful functional components**

`useState` & `useEffect` are the most common **hooks**

### Building Custom Hook

for example we have a function called `Toggler()` which we change a state from `true` to `false` and `fasle` to `true`.
So, we will create a `custom Hook` to add this logic to it. and always a custom name of custom Hooks always start with `use`.

`useToggle.js`

```js
import React, { useState } from 'react';

const useToggle(initialVal = false) {
  const [state, setState] = useState(initialVal);
  const toggle = () => {
    setState(!state)
  }
  
  return [state, toggle];
}

export default useToggle;
```

