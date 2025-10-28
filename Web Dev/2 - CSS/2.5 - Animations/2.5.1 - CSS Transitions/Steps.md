---
tags: 
 - css
 - transition
---

### 🧩 What It Does

The `steps()` function creates **discrete jumps** between states instead of smooth interpolation — perfect for **sprite animations, counters, typing effects**, etc.

```css
transition-timing-function: steps(n, direction);
```

- `n` → number of steps
    
- `direction` → optional keyword: `start` or `end`
    

---

### ⚙️ Syntax

```
steps( <number-of-steps>, [ start | end ] )
```

|Parameter|Description|
|---|---|
|🔢 `n`|Number of discrete intervals the transition will be divided into|
|🎯 `start`|Jumps happen **at the start** of each step|
|🕐 `end` _(default)_|Jumps happen **at the end** of each step|

---

### 🎬 How It Works

Example:

```css
.element {
  transition: transform 1s steps(4, end);
}
```

➡ The animation will make **4 distinct jumps** within 1 second, not a smooth curve.

🧮 Think of it as:

- `steps(4, end)` → jump after each ¼ time passes
    
- `steps(4, start)` → jump right away, then wait
    

---

### 📊 Visualization

|Function|Description|
|---|---|
|`steps(5, end)`|5 equal blocks — changes at the **end** of each block|
|`steps(5, start)`|5 equal blocks — changes at the **start** of each block|

Imagine a staircase:  
🪜 “end” means you **step up at the edge**,  
🪜 “start” means you **jump up first** and stand still for the rest of the block.

---

### 💡 Practical Examples

```css
/* Typing effect (instant character reveal) */
.text {
  transition: width 2s steps(20, end);
}

/* Frame-by-frame sprite animation */
.sprite {
  animation: walk 1s steps(6, end) infinite;
}

/* Counter or score tick */
.counter {
  transition: transform 1s steps(10, start);
}
```

---

### 🧠 Tips

- Use `steps()` when you want **sharp transitions**, not smooth easing.
    
- The more steps you set, the **finer** the motion.
    
- Combine with `animation-iteration-count: infinite` for looping sprite effects.
    
- Default is `end`, so `steps(4)` = `steps(4, end)`.
    