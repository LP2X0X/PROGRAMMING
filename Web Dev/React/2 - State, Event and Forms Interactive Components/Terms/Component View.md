---
tags:
  - react
  - term
  - fundamental
---

### 🔹 Component View (Short Summary)

In React, a **component view** refers to the **UI part of a component** — what it **renders on the screen**. It's defined by the **JSX returned** from the component function.

> 🧠 Think of it as **what the user sees**, based on the component’s state and props.

For example:

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>; // 👈 this is the component's "view"
}
```