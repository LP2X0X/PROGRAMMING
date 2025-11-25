---
tags: 
 - react
 - technique
---

HTML **does NOT** have a `defaultValue` property on `<input>` elements. `defaultValue` is a **React-specific** property.

---

# ✅ **HTML (Raw DOM)**

In plain HTML / JavaScript, an `<input>` element has:

- `.value` → the current value
    
- `.defaultValue` → **exists but is not used the same way as React**
    

HTML `<input>` _does_ expose a `defaultValue` property in the **DOM API**, but:

- It is simply the **initial value taken from the HTML attribute `value="..."`**
    
- It does NOT control the element after initial render
    
- It is **not used for two-way binding or state control**
    

Example (HTML):

```html
<input id="age" value="20" />
<script>
  const el = document.getElementById("age");
  console.log(el.defaultValue); // "20"
  console.log(el.value);        // "20"
</script>
```

So yes, **the DOM has `defaultValue`, but it's very low-level and not a feature developers normally use**.

---

# ✅ **React**

React uses:

- `defaultValue` → for **uncontrolled components**
    
- `value` → for **controlled components**
    

Example:

```jsx
<input defaultValue="Hello" />
```

This sets the initial value only, and React does _not_ control it afterward.

---

# 🧠 **Important Distinction**

### In HTML:

`defaultValue` = reflects the initial HTML attribute.

### In React:

`defaultValue` = React’s way to initialize an uncontrolled input.

React did **not invent** the property — it simply exposed it in JSX as an API.

---

# 🔥 Quick Answer

**Yes, input elements in the DOM have a `.defaultValue` property, but the `defaultValue` you use in React is React's controlled/uncontrolled API, not an HTML feature.**
