---
tags:
  - react
  - technique
---

e.preventDefault() in React works the same way it does in plain DOM: it stops the browser’s **default action** for that event so you can substitute your own logic.

---

## **Common uses**
  
### **1. Prevent form submission from reloading the page:**

```jsx
import { useState } from "react";

function MyForm() {
  const [text, setText] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault(); // <- stops the browser from doing full-page submit/reload
    console.log("Submitted value:", text);
    // do whatever (AJAX, state updates, etc.)
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Type something"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

#### **2. Prevent an \<a> link from navigating:**
```jsx
<a
  href="https://example.com"
  onClick={(e) => {
    e.preventDefault(); // stops navigation
    // custom behavior instead
    alert("Link clicked, but navigation prevented");
  }}
>
  Click me
</a>
```

---

### **Notes**

- In React event handlers, e is a **SyntheticEvent** (a cross-browser wrapper). In React 17+ event pooling was removed, so you can safely use e asynchronously; you no longer need e.persist() except if you’re on a much older React version.
- Returning false from an event handler does **not** prevent default behavior in React; you must call e.preventDefault() explicitly.
- If you want to stop the event from bubbling up as well, use e.stopPropagation() in addition to or instead of preventDefault() depending on intent.