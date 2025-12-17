---
tags: 
 - js
 - event
 - phase
---

JavaScript events travel through the DOM in **three phases**:

---

# 1. Capturing Phase (Phase 1)

**Direction:**  
`document → html → body → ... → target parent → target`

**Characteristics:**

- The event moves **down** the DOM tree.
    
- Only listeners registered with `{ capture: true }` run.
    

```js
element.addEventListener("click", handler, { capture: true });
```

**`event.eventPhase === 1`**

---

# 2. Target Phase (Phase 2)

**The event reaches the exact element where it occurred.**

- Listeners on the target run **regardless** of capture or bubble mode.
    
- Both capturing and bubbling listeners can run here.
    

**`event.eventPhase === 2`**

---

# 3. Bubbling Phase (Phase 3)

**Direction:**  
`target → target parent → ... → body → html → document`

**Characteristics:**

- The event moves **up** the DOM tree.
    
- Only listeners registered with default mode (bubbling) run.
    

```js
element.addEventListener("click", handler);
```

**`event.eventPhase === 3`**

---

# Visual Diagram

```
CAPTURING ↓
document
  html
    body
      parent
        target  ← TARGET PHASE →
      parent
    body
  html
document
BUBBLING ↑
```

![[Pasted image 20251209153551.png|center|500]]

---

# Important Notes

### 1. Most events **bubble**, but some do not:

- `focus`, `blur` do NOT bubble.
    
- `input`, `change`, `click`, `submit`, etc. DO bubble.
    

### 2. You can stop propagation:

- `stopPropagation()` → stops movement to other elements
    
- `stopImmediatePropagation()` → also stops other listeners on same element
    

The `event.stopPropagation()` method and its sibling `event.stopImmediatePropagation()` can also be called on the capturing phase. Then not only the futher capturing is stopped, but the bubbling as well.

In other words, normally the event goes first down (“capturing”) and then up (“bubbling”). But if `event.stopPropagation()` is called during the capturing phase, then the event travel stops, no bubbling will occur.

### 3. Event delegation depends on **bubbling**

```js
list.addEventListener("click", e => {
  if (e.target.matches("li")) {
    // ...
  }
});
```

### 4. To remove the handler, `removeEventListener` needs the same phase

If we `addEventListener(..., true)`, then we should mention the same phase in `removeEventListener(..., true)` to correctly remove the handler.

### 5. Listeners on the same element and same phase run in their set order

If we have multiple event handlers on the same phase, assigned to the same element with `addEventListener`, they run in the same order as they are created:

```javascript
elem.addEventListener("click", e => alert(1)); // guaranteed to trigger first
elem.addEventListener("click", e => alert(2));
```

---

# Summary Table

| Phase     | eventPhase | Direction        | Listeners Active      |
| --------- | ---------- | ---------------- | --------------------- |
| Capturing | 1          | Parents → target | capture:true          |
| Target    | 2          | At target        | both capture + bubble |
| Bubbling  | 3          | target → parents | bubble (default)      |