---
tags: 
 - js
 - method
 - event
---

`event.stopPropagation()` prevents an event from continuing to travel through the DOM after the current listener finishes.

### What it stops

- **Stops bubbling** (child → parent → document)
    
- **OR stops capturing** if used during the capture phase
    

### What it does _not_ stop

- It **does NOT** stop the default browser action  
    (use `preventDefault()` for that)
    
- It **does NOT** stop other listeners on the same element  
    (use `stopImmediatePropagation()` for that)
    

---

# Example (Stops Bubbling)

```js
child.addEventListener("click", e => {
  e.stopPropagation();
});

parent.addEventListener("click", () => {
  console.log("Not reached");
});
```

Clicking the child triggers only the child’s handler.

---

# When to use it

- Avoid triggering parent handlers (e.g., nested buttons, menus)
    
- Prevent delegation handlers from catching the event
    
- Control multi-level event flows
    

---

# Variants

### **stopPropagation()**

Stops event movement to other elements.

### **stopImmediatePropagation()**

Stops:

1. other listeners on **same element**
    
2. bubbling/capturing
    