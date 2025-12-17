---
tags: 
 - js
 - term
 - event
---

The **Event object** (often named `event` or `e`) is automatically passed to every event handler in JavaScript. It contains metadata about the event, the element that triggered it, the event flow position, input details, and control methods such as `preventDefault()`.

The base type is **Event**, but depending on the event, more specialized event types are used (e.g., `MouseEvent`, `KeyboardEvent`, `InputEvent`, `SubmitEvent`).

---

# 1. How You Receive the Event Object

```js
element.addEventListener("click", function (event) {
  console.log(event);
});
```

The browser injects it automatically.

---

# 2. Core Properties of `Event`

These are available on all event types.

### **event.type**

String name of the event.

```js
event.type; // "click"
```

### **event.target**

The **actual element** that triggered the event.

### **event.currentTarget**

The element whose listener is currently running (may differ from target during bubbling).

```ad-note
Note the differences from `this` (=`event.currentTarget`):

- `event.target` – is the “target” element that initiated the event, it doesn’t change through the bubbling process.
- `this` – is the “current” element, the one that has a currently running handler on it.
```

### **event.eventPhase**

Indicates the phase:

- 0: none
    
- 1: capturing
    
- 2: at target
    
- 3: bubbling
    

### **event.bubbles**

Boolean — does the event bubble?

### **event.cancelable**

Boolean — can `preventDefault()` be applied?

### **event.defaultPrevented**

True if `preventDefault()` was called.

### **event.timeStamp**

When the event occurred (in ms since page load).

---

# 3. Core Methods

### **event.preventDefault()**

Prevents browser default behavior.

Used for:

- stopping form submission
    
- stopping link navigation
    
- preventing scrolling in touch events
    

### **event.stopPropagation()**

Stops the event from bubbling to parent elements.

### **event.stopImmediatePropagation()**

Stops:

1. bubbling
    
2. **other listeners on the same element**
    

### **event.composedPath()**

Returns the chain of elements through which the event traveled.

---

# 4. Specialized Event Types

Different events have extended properties. The Event object becomes a subclass depending on event category.

---

# 4.1. Mouse Events (e.g., click, mousedown)

Type: **MouseEvent**

Key properties:

- `event.clientX`, `event.clientY` (viewport coordinates)
    
- `event.pageX`, `event.pageY`
    
- `event.button` (0=left, 1=middle, 2=right)
    
- `event.altKey`, `event.shiftKey`, `event.ctrlKey`
    
- `event.relatedTarget` (for mouseover/mouseout)
    

---

# 4.2. Keyboard Events (keydown, keyup)

Type: **KeyboardEvent**

Key properties:

- `event.key` (e.g., "a", "Enter")
    
- `event.code` ("KeyA", "Enter")
    
- `event.repeat`
    
- modifier keys: `altKey`, `shiftKey`, `ctrlKey`, `metaKey`
    

---

# 4.3. Input Events (input, change)

Type: **InputEvent**

Key properties:

- `event.data` (inserted text)
    
- `event.inputType` (e.g., "insertText")
    
- `event.target.value`
    

---

# 4.4. Submit Events

Type: **SubmitEvent**

Key properties:

- `event.submitter` (the button used to submit the form)
    

---

# 4.5. Touch Events (mobile)

Type: **TouchEvent**

Key properties:

- `event.touches`
    
- `event.changedTouches`
    
- `event.targetTouches`
    

Each contains Touch objects with `clientX`, `clientY`, etc.

---

# 4.6. Focus Events

Type: **FocusEvent**

Key properties:

- `event.relatedTarget` (element losing focus or receiving focus)
    

---

# 5. Event Propagation (How event travels)

Phases:

1. **Capturing** (document → target)
    
2. **At Target**
    
3. **Bubbling** (target → document)
    

Event object tracks this in:

- `event.eventPhase`
    
- `event.target`
    
- `event.currentTarget`
    

You can register in capture phase:

```js
element.addEventListener("click", handler, { capture: true });
```

---

# 6. Event Delegation and Event.target

Event delegation uses `event.target` to identify which child triggered the event:

```js
parent.addEventListener("click", e => {
  if (e.target.matches(".delete-btn")) {
    // clicked a delete button inside parent
  }
});
```

`currentTarget` always refers to the parent with the listener.

---

# 7. Synthetic Events in React (for comparison)

React uses **SyntheticEvent** — a wrapper around native events, but with identical API.

Important:

- In React 18, the synthetic event no longer pools
    
- Values persist across async calls
    
- Still behaves similarly to DOM events
    

---

# 8. Summary Table

| Item                               | Description                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `event.target`                     | actual element that fired the event                      |
| `event.currentTarget`              | element whose listener is running                        |
| `event.type`                       | event name                                               |
| `event.bubbles`                    | event supports bubbling                                  |
| `event.cancelable`                 | can call preventDefault()                                |
| `event.preventDefault()`           | blocks default browser action                            |
| `event.stopPropagation()`          | stops bubbling                                           |
| `event.stopImmediatePropagation()` | stops bubbling + other handlers                          |
| `event.timeStamp`                  | time event occurred                                      |
| Specialized types                  | MouseEvent, KeyboardEvent, InputEvent, SubmitEvent, etc. |
