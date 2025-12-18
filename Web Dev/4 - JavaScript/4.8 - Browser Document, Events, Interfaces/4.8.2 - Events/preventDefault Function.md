---
tags: 
 - js
 - function
 - event
---

`event.preventDefault()` is a method on the DOM `Event` object that **cancels the default browser behavior** associated with a particular event.

It does not stop the event from propagating; it only stops the browser from performing what it normally would do for that event.

---

# **1. Why It Exists**

Many DOM events come with built-in browser actions. Examples:

- Clicking a link → navigates to another URL.
    
- Submitting a form → reloads page and sends data.
    
- Pressing a key in an input → edits text.
    
- Right-clicking → opens context menu.
    
- Drag-and-drop → moves selected elements.
    
- Selecting text → highlights text.
    

`preventDefault()` allows you to **override** these built-in actions.

---

# **2. Syntax**

```js
event.preventDefault();
```

---

# **3. Common Use Cases**

### **A. Prevent form submission**

Normally a form submit reloads/resets the page.

```js
form.addEventListener("submit", (e) => {
  e.preventDefault();
  // handle custom submission logic
});
```

### **B. Prevent link navigation**

```js
document.querySelector("a").addEventListener("click", (e) => {
  e.preventDefault();
});
```

### **C. Disable context menu**

```js
window.addEventListener("contextmenu", (e) => e.preventDefault());
```

### **D. Prevent text selection or drag**

```js
element.addEventListener("mousedown", (e) => e.preventDefault());
```

### **E. Prevent checkbox from toggling**

```js
checkbox.addEventListener("click", (e) => e.preventDefault());
```

### **F. Prevent scrolling with keyboard events**

```js
window.addEventListener("keydown", (e) => {
  if (e.key === "ArrowDown") e.preventDefault();
});
```

---

# **4. Important Notes**

### **A. It does NOT stop propagation**

`preventDefault()` does not stop the event from bubbling or capturing.

To stop propagation, use:

```js
event.stopPropagation();
```

To do both:

```js
event.preventDefault();
event.stopPropagation();
```

---

### **B. Only works on “cancelable” events**

Not all events allow preventing default action.

Check with:

```js
event.cancelable; // true or false
```

---

### **C. React synthetic events behave the same**

```jsx
function handleClick(e) {
  e.preventDefault();
}
```

React’s synthetic `preventDefault()` mirrors DOM behavior.

---

# **Summary (Short Note)**

`event.preventDefault()` stops the browser’s default action for an event—for example, preventing form submission, blocking navigation, disabling context menus, or stopping text selection. It does not stop propagation; it only cancels the built-in browser behavior for cancelable events.