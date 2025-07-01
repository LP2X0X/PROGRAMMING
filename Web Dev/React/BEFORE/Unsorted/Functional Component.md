---
tags: react, term, fundamental
---

In React, a **functional component** is just a JavaScript function that returns JSX. It’s the simplest and most modern way to create components.

---

## **✅ Basic Functional Component**

```js
function Welcome() {
  return <h1>Hello, world!</h1>;
}
```

Or using arrow function syntax:

```js
const Welcome = () => {
  return <h1>Hello, world!</h1>;
};
```

You use it like this:

```
<Welcome />
```

---

## **📦 With props**

```js
const Welcome = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// Usage
<Welcome name="Alice" />
```

---

## **🧠 With useState and useEffect (React Hooks)**

```js
import { useState, useEffect } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Component mounted or updated');
  }, [count]);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
};
```

---

## **🧩 Organizing Components**

React functional components are often organized into files like:

```
/src
  /components
    Welcome.jsx
    Counter.jsx
  App.jsx
  main.jsx
```

Then imported like:

```js
import Welcome from './components/Welcome';
```