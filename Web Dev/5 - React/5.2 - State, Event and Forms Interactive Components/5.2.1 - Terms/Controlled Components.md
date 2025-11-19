---
tags:
  - react
  - technique
  - fundamental
---

A **controlled component** in React is a form element (like `<input>`, `<textarea>`, or `<select>`) whose **value is controlled by React state** — not by the DOM.

> This means that the value of the form element is set and updated through React state, making React the "single source of truth" for the form data.

---

## 🧠 Think of it like this:

> The **source of truth** for the input’s value is **React's state**, not the browser.

---

## ✅ Example: Controlled Input

```jsx
import { useState } from "react";

function MyForm() {
  const [name, setName] = useState("");

  return (
    <input
      type="text"
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

- `value={name}` → the input **always reflects** the `name` state.
    
- `onChange` → any user typing updates the state.
    
- So the value is fully under React's control.
    

---

## 🆚 Controlled vs Uncontrolled

|Feature|Controlled|Uncontrolled|
|---|---|---|
|Value stored in|**React state**|DOM|
|React updates?|✅ Yes|❌ No|
|Need `value` + `onChange`?|✅ Yes|❌ Optional|
|Use case|Forms where you need validation, dynamic behavior, or React control|Quick forms, less logic|

---

## 🛠 When to Use Controlled Components?

- You need to:
    
    - Validate input while typing
        
    - Disable/enable fields dynamically
        
    - Reflect input elsewhere in the UI
        
    - Submit form data from state
        

---

## 🧪 Example with a form:

```jsx
function SignupForm() {
  const [email, setEmail] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Submitted email:", email);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## ✅ Summary

- A **controlled component** has its value controlled by **React state**.
    
- It's best for situations where you need full control of form inputs.
    
- Requires both `value` and `onChange` props.
    