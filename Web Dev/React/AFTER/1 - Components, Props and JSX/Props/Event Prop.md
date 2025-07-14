---
tags: react, prop, fundamental
---

In React, **event-related props** are props that handle user interactions — like clicks, key presses, form submissions, and more. These are passed to JSX elements (or custom components) to define behavior when events happen.

---

### **✅ Common React Event Props**

|**Event**|**Prop Name**|**Example**|
|---|---|---|
|Click|onClick|\<button onClick={handleClick} />|
|Change (input)|onChange|\<input onChange={handleChange} />|
|Submit|onSubmit|\<form onSubmit={handleSubmit} />|
|Mouse enter/leave|onMouseEnter, onMouseLeave|\<div onMouseEnter={...} />|
|Key press/down|onKeyDown, onKeyUp, onKeyPress|\<input onKeyDown={...} />|
|Focus/Blur|onFocus, onBlur|\<input onFocus={...} />|

---

### **🧠 Syntax Rules**

- Event prop names use **camelCase** (e.g., onClick, not onclick)
    
- The value is a **function reference** (not a string like in HTML)
    

```jsx
function handleClick() {
  alert("Button clicked!");
}

function App() {
  return <button onClick={handleClick}>Click Me</button>;
}
```

```ad-note
We usually define the event handler function in the component body (but outside the JS block bracket).
```

---

### **✅ Event Handler with Parameters**

```jsx
<button onClick={() => handleClick("Hello")}>Click</button>
```

You can use an arrow function to pass arguments.

````ad-warning
Do not use function call in it like this:
```jsx
<button onClick={handleClick("Hello")}>Click</button>
```
Because when React first initialize this, it will then see this code and will immediately execute it (in this case the function will be call since the syntax above is a function call syntax).
````

---

### **✅ Event Object**

React passes a **synthetic event object** to your handler:

```jsx
function handleChange(event) {
  console.log(event.target.value);
}
```

---

### **✅ Using in Custom Components**

You can forward event props too:

```jsx
function CustomButton({ onClick }) {
  return <button onClick={onClick}>Click</button>;
}
```