---
tags: 
 - css
 - event 
 - transition
---

### 🧩 What It Is

The `transitionend` event fires in JavaScript when a **CSS transition finishes** on an element.  
It lets you run logic **after** a transition completes — such as removing classes, starting another animation, or cleaning up DOM state.

---

### ⚙️ Syntax

```js
element.addEventListener('transitionend', callback);
```

or inline with options:

```js
element.addEventListener('transitionend', (event) => {
  // handle event
}, { once: true });
```

---

### 🧾 Event Object Properties

|Property|Type|Description|
|---|---|---|
|`event.propertyName`|string|The CSS property that finished transitioning|
|`event.elapsedTime`|number|Duration in seconds (not including delay)|
|`event.pseudoElement`|string|Name of pseudo-element (e.g. `::before`) or empty string|

---

### 🧠 Key Points

- Fires **once for each transitioning property**.  
    → If you transition `opacity` and `transform`, it fires twice.
    
- Doesn’t fire if:
    
    - The transition was **canceled** (e.g. property changed again before finishing)
        
    - The property didn’t actually change value
        
- You can use `{ once: true }` to auto-remove the listener after it runs.
    

---

### 💡 Example

```html
<div class="box"></div>
```

```css
.box {
  width: 100px;
  height: 100px;
  background: skyblue;
  transition: transform 0.5s ease;
}
.box.active {
  transform: translateX(200px);
}
```

```js
const box = document.querySelector('.box');

box.addEventListener('click', () => {
  box.classList.add('active');
});

box.addEventListener('transitionend', (e) => {
  console.log(`Transition done: ${e.propertyName}`);
  box.style.background = 'limegreen';
});
```

🧩 Result:

- When the box finishes moving, the console logs
    
- Background changes to green after the transition ends
    

---

### ⚠️ Common Gotchas

- If multiple properties transition, wrap your code:
    
    ```js
    if (e.propertyName === 'transform') {
      // only run once for transform
    }
    ```
    
- If you need the **whole group of transitions** to finish, use a counter or `Promise.all()` pattern.
    

---

### 🚀 Pro Tip

To run something after _any_ transition, even when multiple properties animate:

```js
const onEnd = (e) => {
  if (e.target === e.currentTarget && e.propertyName === 'opacity') {
    // your logic here
  }
};
element.addEventListener('transitionend', onEnd);
```