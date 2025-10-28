---
tags: 
 - css
 - keyframe
---

### 🧩 What It Is

The `@keyframes` rule defines the **stages of an animation** — how an element’s styles change **over time**.  
Used together with the `animation` property, it creates smooth or step-based motion without JavaScript.

---

### ⚙️ Basic Syntax

```css
@keyframes animationName {
  from { /* starting styles */ }
  to   { /* ending styles */ }
}
```

Or with percentages:

```css
@keyframes animationName {
  0%   { /* start */ }
  50%  { /* halfway */ }
  100% { /* end */ }
}
```

---

### 🧱 Applying the Animation

```css
.selector {
  animation-name: animationName;
  animation-duration: 2s;
  animation-timing-function: ease;
  animation-delay: 0s;
  animation-iteration-count: 1;
  animation-direction: normal;
  animation-fill-mode: none;
  animation-play-state: running;
}
```

📦 Or shorthand:

```css
animation: animationName 2s ease-in-out 0.5s infinite alternate;
```

---

### 📊 Animation Properties Reference

|Property|Description|Example|
|---|---|---|
|🎞️ `animation-name`|The keyframes identifier|`slideIn`|
|⏱️ `animation-duration`|How long the animation lasts|`1s`, `500ms`|
|🧮 `animation-timing-function`|How progress changes over time|`ease`, `linear`, `steps(4)`, `cubic-bezier(0.25,1,0.5,1)`|
|⏳ `animation-delay`|Wait time before starting|`0.5s`|
|🔁 `animation-iteration-count`|Number of loops|`1`, `3`, `infinite`|
|🔄 `animation-direction`|Playback direction|`normal`, `reverse`, `alternate`, `alternate-reverse`|
|🧍 `animation-fill-mode`|How styles persist before/after|`none`, `forwards`, `backwards`, `both`|
|▶️ `animation-play-state`|Paused or running|`running`, `paused`|

---

### 💡 Example

```css
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-30px); }
  60% { transform: translateY(-15px); }
}

.ball {
  animation: bounce 2s ease infinite;
}
```

⚽ The element jumps up and down continuously like a bouncing ball.

---

### 🧠 Tips & Tricks

- Combine multiple animations:
    
    ```css
    animation: slide 1s ease, fade 1s ease;
    ```
    
- Use `animation-delay` with **negative values** to start midway through an animation.
    
- `forwards` keeps the **final frame** visible after finishing.
    
- Use `animation-play-state: paused` to freeze an animation mid-frame.
    

---

### 🧩 Example with Multiple Keyframes

```css
@keyframes colorAndMove {
  0%   { background: red; transform: translateX(0); }
  50%  { background: yellow; transform: translateX(50px); }
  100% { background: lime; transform: translateX(100px); }
}

.box {
  animation: colorAndMove 3s cubic-bezier(0.25, 1, 0.5, 1) infinite alternate;
}
```

---

### ⚠️ Common Gotchas

- You **must** use a unique name for each `@keyframes` in the same scope.
    
- Transitions (`transition`) only animate **to/from a defined state**,  
    but `@keyframes` can define **many intermediate states**.
    
- Browsers won’t animate non-animatable properties (like `display`).
    

---

### 🧮 Bonus: Event Hooks (JS)

```js
element.addEventListener('animationend', () => {
  console.log('Animation finished!');
});
```

Other events:

- `animationstart`
    
- `animationiteration`
    
- `animationend`
    