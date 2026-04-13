---
tags: 
 - css
 - animation
 - transition
---

The `transition` property is a shorthand in CSS used to control the speed and manner in which a property changes from one state to another (e.g., from a blue background to a red one on hover).

Without transitions, property changes happen **instantly**. Transitions allow these changes to occur over a **duration**.

---

## 1. The Shorthand Syntax

The `transition` property combines four sub-properties into one line:

```css
/* property | duration | timing-function | delay */
.btn {
  transition: background-color 0.3s ease-in 0.1s;
}
```

1. **`transition-property`**: The name of the CSS property you want to animate (e.g., `opacity`, `transform`, `width`). Use `all` to transition everything that changes.
    
2. **`transition-duration`**: How long the animation takes (e.g., `200ms`, `0.5s`).
    
3. **`transition-timing-function`**: The "speed curve" of the transition (how it starts and ends).1
    
4. **`transition-delay`**: How long to wait before the transition starts.
    

---

## 2. Timing Functions (Easing)

This defines the "vibe" of the movement. Common values include:

- **`ease`**: (Default) Starts slow, speeds up, ends slow.
    
- **`linear`**: Same speed from start to finish.
    
- **`ease-in`**: Starts slow, ends fast.2
    
- **`ease-out`**: Starts fast, ends slow.
    
- **`cubic-bezier(n, n, n, n)`**: Allows you to create custom physics-based curves.
    

---

## 3. Transitioning Multiple Properties

If you want different durations for different properties, separate them with a comma:

```css
.card {
  transition: transform 0.3s ease, opacity 0.5s linear;
}
```

---

## 4. Best Practices and Performance

### A. Only Animate "Cheap" Properties

For a smooth 60fps experience, try to stick to animating **`transform`** and **`opacity`**.

- **Cheap:** `opacity`, `scale`, `rotate`, `translate`.
    
- **Expensive:** `width`, `height`, `margin`, `top`. These trigger "Layout Reflow," which can cause stuttering on mobile devices.
    

### B. The "Back to Normal" Rule

Define the transition on the **base element**, not just the `:hover` state.

- If you put it on `:hover`, the transition works when the mouse enters, but it will snap back instantly when the mouse leaves.
    
- Putting it on the base class ensures the transition works in **both directions**.
    

```css
/* ✅ DO THIS */
.box {
  background: blue;
  transition: background 0.5s;
}
.box:hover {
  background: red;
}
```

---

## 5. Transitioning to `display: none`?

A common "gotcha" is that you **cannot** transition the `display` property. If you change from `display: block` to `display: none`, the element disappears instantly, ignoring the duration.

**The Workaround:** Transition `opacity` and `visibility` instead:

```css
.dropdown {
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
}

.parent:hover .dropdown {
  opacity: 1;
  visibility: visible;
}
```

---

## 6. Summary Table

| **Property**                     | **Description**    | **Common Values**             |
| -------------------------------- | ------------------ | ----------------------------- |
| **`transition-property`**        | What to animate    | `all`, `transform`, `opacity` |
| **`transition-duration`**        | How long it takes  | `0.3s`, `500ms`               |
| **`transition-timing-function`** | The speed curve    | `ease-in-out`, `cubic-bezier` |
| **`transition-delay`**           | Pause before start | `0.1s`, `0s`                  |