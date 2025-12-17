---
tags: 
 - js
 - term
 - event
---

# 1. What is an Event Handler?

An **event handler** is a function that executes in response to a specific event such as:

- click
    
- submit
    
- keydown
    
- input
    
- change
    
- load
    
- DOMContentLoaded
    
- mouseover
    
- focus / blur
    

JavaScript interacts with the browser’s event system through these handlers.

Example:

```js
button.addEventListener("click", function () {
  console.log("Button clicked");
});
```

Here, the function is the _event handler_ for the `click` event.

---

# 2. Three Ways to Register Event Handlers

## A. **Inline HTML event handlers** (discouraged)

```html
<button onclick="alert('Hello')">Click</button>
```

Why discouraged:

- mixes HTML and JS
    
- less maintainable
    
- harder to debug
    
- no access to modern patterns (e.g., `event.preventDefault()` easily)
    

```ad-note
When the browser reads the attribute, it creates a handler function with body from the attribute content.
```

---

## B. **Using DOM element properties** (simple but limited)

```js
button.onclick = function () {
  console.log("clicked");
};
```

Characteristics:

- Easy to use
    
- But **only one handler per event**—assigning again overwrites the previous one
    
- No event options (capture, passive, once)
    

---

## C. **Using addEventListener()** (best practice)

```js
button.addEventListener("click", handleClick);

function handleClick(event) {
  console.log("clicked");
}
```

Advantages:

- Multiple handlers for the same event
    
- Event options support:
    
    - `{ once: true }`
        
    - `{ capture: true }`
        
    - `{ passive: true }`
        
- Works for all modern DOM APIs
    

This is the recommended approach.

---

# 3. The Event Object (`event`)

Every handler receives an `Event` object automatically.

```js
element.addEventListener("click", function (event) {
  console.log(event.type); // "click"
  console.log(event.target); // element that triggered the event
});
```

Common properties:

- `event.type`
    
- `event.target`
    
- `event.currentTarget`
    
- `event.key` (for keyboard)
    
- `event.clientX` / `clientY` (mouse)
    
- `event.preventDefault()`
    
- `event.stopPropagation()`
    

---

# 4. Important Event Handler Methods

## A. `event.preventDefault()`

Prevents browser’s default behavior.

Examples:

- stop form submission
    
- prevent a link from navigating
    

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();
  console.log("Form prevented");
});
```

---

## B. `event.stopPropagation()`

Stops event from bubbling up the DOM tree.

```js
child.addEventListener("click", e => {
  e.stopPropagation();
});
```

---

## C. `event.stopImmediatePropagation()`

Stops _all_ handlers on the same element from running.

---

# 5. Event Flow: Capture → Target → Bubble

When an event occurs:

1. **Capture phase** (top → target)
    
2. **Target phase** (on element)
    
3. **Bubble phase** (target → top)
    

Default is bubbling.

```js
element.addEventListener("click", handler, { capture: true });
```

---

# 6. Removing Event Handlers

Only possible with `addEventListener()`:

```js
element.addEventListener("click", handler);
element.removeEventListener("click", handler);
```

You must pass the **same function reference**, not an inline function.

---

# 7. Anonymous vs Named Handlers

Anonymous:

```js
button.addEventListener("click", () => console.log("clicked"));
```

Named:

```js
function handleClick() {
  console.log("clicked");
}
button.addEventListener("click", handleClick);
```

Named handlers:

- can be reused
    
- can be removed
    
- are better for debugging
    

---

# 8. Event Delegation (advanced but essential)

Attach a single listener on a parent to handle many children events.

```js
document.body.addEventListener("click", (event) => {
  if (event.target.matches(".delete-btn")) {
    console.log("Delete clicked");
  }
});
```

Benefits:

- fewer listeners
    
- better performance
    
- handles dynamically added elements
    

---

# 9. Summary

| Concept                   | Description                     |
| ------------------------- | ------------------------------- |
| Event handler             | Function responding to an event |
| Inline method             | HTML-based, discouraged         |
| Property method           | Simple, one handler only        |
| `addEventListener()`      | Modern, flexible, recommended   |
| `event.preventDefault()`  | Stops default behavior          |
| `event.stopPropagation()` | Stops bubbling                  |
| Capture phase             | Top → target                    |
| Bubble phase              | Target → top                    |
| Event delegation          | Parent handles child events     |
