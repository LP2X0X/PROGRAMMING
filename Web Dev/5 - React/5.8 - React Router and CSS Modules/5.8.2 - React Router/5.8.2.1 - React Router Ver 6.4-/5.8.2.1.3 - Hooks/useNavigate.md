---
tags: 
 - react
 - router
 - hook
---

`useNavigate` is a hook that lets you **navigate programmatically** in React Router.

You use it when you want to change pages **inside a function**, not by clicking a `<Link>`.

---

## ✅ **Basic Usage**

```js
import { useNavigate } from "react-router-dom";

function MyComponent() {
  const navigate = useNavigate();

  function goToHome() {
    navigate("/home");
  }

  return <button onClick={goToHome}>Go Home</button>;
}
```

---

## 🧭 **Navigate with history behaviors**

### **1. Push (default)**

Adds a new entry to browser history:

```js
navigate("/login");
```

### **2. Replace current history entry**

```js
navigate("/login", { replace: true });
```

Useful after login, logout, form submission, etc.

---

## ⏳ **Navigate backward or forward**

```js
navigate(-1); // go back
navigate(1);  // go forward
```

---

## 📦 **Navigate with params or state**

```js
navigate("/profile", {
  state: { from: "checkout" }
});
```

In the next page:

```js
const location = useLocation();
console.log(location.state.from);
```

---

## 🔐 **Common use cases**

- Redirect after form submission
    
- Redirect after login/logout
    
- Navigate from inside async functions
    
- Navigate after a mutation (e.g., deleting an item)
    