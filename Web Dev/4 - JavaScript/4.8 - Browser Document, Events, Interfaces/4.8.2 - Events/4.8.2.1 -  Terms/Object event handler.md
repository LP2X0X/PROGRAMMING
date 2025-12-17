---
tags: 
 - js
 - term
---

In JavaScript, instead of passing a function as an event listener, you can pass an **object** that implements a method named **`handleEvent`**.

### How it works

```js
const handler = {
  handleEvent(event) {
    console.log("Event type:", event.type);
  }
};

elem.addEventListener("click", handler);
```

When the event fires, the browser automatically calls:

```js
handler.handleEvent(event);
```

### Why use it

- Lets you group related event handling logic inside an object.
    
- Allows **multiple event types** using the same object, with internal routing:
    

```js
const handler = {
  handleEvent(e) {
    if (e.type === "mousedown") this.onDown(e);
    if (e.type === "mouseup") this.onUp(e);
  },
  onDown(e) { /* ... */ },
  onUp(e) { /* ... */ }
};
```

```js
<button id="elem">Click me</button>

<script>
  class Menu {
    handleEvent(event) {
      switch(event.type) {
        case 'mousedown':
          elem.innerHTML = "Mouse button pressed";
          break;
        case 'mouseup':
          elem.innerHTML += "...and released.";
          break;
      }
    }
  }

  let menu = new Menu();

  elem.addEventListener('mousedown', menu);
  elem.addEventListener('mouseup', menu);
</script>
```

- Useful when managing state across events (drag, gestures, long sequences).
    

### Key Notes

- `handleEvent` is the _only required method_.
    
- The object becomes the listener.
    
- Works with `addEventListener` and `removeEventListener`.
    
- `this` inside `handleEvent` refers to the object itself (unlike functions where `this` varies).
    