---
tags:
  - react
  - term
  - fundamental
---

### 🔷 What Is **State** in React?

- In React, **state** is a **built-in object** that stores data **specific to a component** and controls how that component **behaves or appears**.
- When state changes (useState), React **re-render** the component that owns that state.
- If that state is passed as a prop to child components, those children are **also re-rendered**.

![[Screenshot 2025-07-14 at 19.30.24.png|center|500]]

---

### ✅ Example: A Counter Using State

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // ✅ state declaration

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

### ⚙️ The mechanics of State

![[Screenshot 2025-07-10 at 03.59.07.png|center|500]]

---

### ⏰ State updates are asynchronous

- When you call a state setter like setCount(count + 1), React:
	- **Does not update count immediately in the same function call**.
	- Instead, it **schedules** the update.

```js
function Example() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // still the old value!
  }
}
```

- The reason is [[State updates are batched|state updates are batched]].

---

### 📦 `useState()` Hook

* `useState()` is a **hook** that lets you use state in **function components**.
* It returns an **array**:

  * `count`: the current state value
  * `setCount`: the function to update it

```js
const [value, setValue] = useState(initialValue);
```

---

### 🧠 Key Concepts

| Concept                | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| **State is local**     | Each component manages its own state.                                           |
| **Triggers re-render** | Updating state causes the component to re-render.                               |
| **Async update**       | State updates are **not immediate**, so rely on callbacks or effects if needed. |

---
 
### ✅ Rules for Creating and Using State in React

When creating state in React using `useState`, follow these key rules:

---

### 1. **Only Call `useState` at the Top Level**

✅ **Do:**

```jsx
function MyComponent() {
  const [count, setCount] = useState(0); // top-level
  // other logic
}
```

❌ **Don’t do:**

```jsx
if (someCondition) {
  const [count, setCount] = useState(0); // ❌ invalid
}
```

> 🔸 State hooks must always run in the same order on every render.

---

### 2. **Only Call `useState` in React Functions**

✅ Valid places to call `useState`:

* Inside a **React function component**
* Inside a **custom hook**

❌ Invalid:

```js
function notAComponent() {
  const [state, setState] = useState(); // ❌ Error
}
```

---

### 3. **Provide an Initial Value**

```jsx
const [count, setCount] = useState(0); // number
const [name, setName] = useState('');  // string
const [items, setItems] = useState([]); // array
```

> The initial value determines the state’s data type.

```jsx
function MyComponent() {
  const [count, setCount] = React.useState(); // no default
  return <p>{count}</p>;
}
```

- Here, `count` starts as **`undefined`**.
- Rendering works fine (React can render `undefined` → it just shows nothing).
- But if you try `count + 1`, you’ll get **NaN**.
- If you expect a string or object, operations on it may fail.

✅ Best practice: Always provide a sensible default value in `useState`.

```jsx
const [count, setCount] = React.useState(0); // number
const [text, setText] = React.useState("");  // string
const [items, setItems] = React.useState([]); // array
const [user, setUser] = React.useState(null); // object that may be absent
```

```ad-note
- Why it matters:
	- Prevents runtime errors (accessing properties of `undefined`).
	- Makes the component predictable.
	- Improves readability → you can see expected data types at a glance.
```

---

### 4. **Never Mutate State Directly**

❌ Don’t do this:

```js
state.value = 42; // ❌ wrong
```

✅ Use the updater function:

```js
setState({ value: 42 }); // ✅
```

- This is also appliable to object in which to update it, we must create a new object with modification instead of changing only necessary properties.

---

### 5. **State Updates Are Asynchronous**

```jsx
setCount(count + 1);
// count is still the old value immediately after this line
```

> Use updater form if needed:

```jsx
setCount(prevCount => prevCount + 1);
```

```ad-note
Use the above solution when you needed to update state based on the current state.
The reason is because React batches updates to improve performance and avoid unnecessary re-renders. But batching also means **you can’t rely on immediate state changes in the same function scope**.
```

---

### 6. **Each `useState` Call Is Independent**

```jsx
const [name, setName] = useState('');
const [age, setAge] = useState(0);
// These are two separate pieces of state
```