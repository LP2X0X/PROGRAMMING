---
tags: react, technique, fundamental
---

### 🔷 Conditional Rendering in React

Conditional rendering in React means **showing or hiding components or elements** based on a condition (like `if`, `true/false`, or a state value).

Here are the most common patterns:

---

### ✅ 1. **Using `if` (outside JSX)**

- Use this when you want to return an entirely different component.

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  return <h1>Please log in.</h1>;
}
```

---

### ✅ 2. **Ternary Operator (inline inside JSX)**

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <h1>{isLoggedIn ? 'Welcome back!' : 'Please log in.'}</h1>
  );
}
```

---

### ✅ 3. **Short-circuiting with `&&`**

Only renders the right side if the left side is `true`.

```jsx
function Notification({ hasMessages }) {
  return (
    <div>
      {hasMessages && <p>You have new messages!</p>}
    </div>
  );
}
```

---

### ✅ 4. **Conditional Block with Variable**

```jsx
function App({ isLoading }) {
  let content;
  if (isLoading) {
    content = <p>Loading...</p>;
  } else {
    content = <p>Data loaded!</p>;
  }

  return <div>{content}</div>;
}
```

---

### ✅ 5. **Return `null` to Render Nothing**

```jsx
function Warning({ show }) {
  if (!show) return null;

  return <p className="warning">Warning!</p>;
}
```

---

### ✅ 6. **Optional Chaining + Ternary (with objects)**

```jsx
const user = { name: 'Long' };

return (
  <p>Hello, {user ? user.name : 'Guest'}</p>
);
```

---

### 💡 Pro Tip:

* Keep JSX clean by **avoiding complex logic inside JSX**.
* Use helper functions or variables if the condition is long.