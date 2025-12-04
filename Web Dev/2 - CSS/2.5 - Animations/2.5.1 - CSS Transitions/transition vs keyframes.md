---
tags: 
 - css
 - animation
 - note
 - distinguish
---

# 🔥 CSS Transition vs CSS Keyframes — The Difference

## ✅ **1. Transition**

**What it is:**  
A transition animates **between two states** — usually triggered by an event (hover, focus, class change).

**Key characteristics:**

- Needs a _start_ and _end_ value.
    
- Requires a **trigger** (hover, adding/removing a class, JS changes).
    
- Simple, single-step animations.
    
- Cannot loop.
    

**Example:**

```css
.box {
  transition: transform 0.3s ease;
}
.box:hover {
  transform: scale(1.2);
}
```

➡️ Animation only happens when user hovers.

**Use transitions when:**

- Hover effects
    
- Button press animations
    
- Showing/hiding elements smoothly
    
- One-step property changes
    

---

## ✅ **2. Keyframes (@keyframes animation)**

**What it is:**  
A keyframe animation defines **multiple stages** of animation, not just start/end.

**Key characteristics:**

- Does **not** require a trigger (can run automatically).
    
- Can loop (`infinite`).
    
- Allows **complex, multi-step** animations.
    
- Can animate many stages (0%, 50%, 100%, etc.).
    
- Can run continuously.
    

**Example:**

```css
@keyframes spin {
  0%   { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.box {
  animation: spin 1s linear infinite;
}
```

➡️ Runs on its own, loops forever.

**Use keyframes when:**

- Loaders/spinners
    
- Repeating animations
    
- Multi-step movement (bounce, pulse, shake…)
    
- Automatic animations without user interaction
    
- Complex timelines
    

---

# 🧩 Quick Comparison Table

| Feature              | `transition`    | `@keyframes` animation           |
| -------------------- | --------------- | -------------------------------- |
| Needs trigger        | ✔️ Yes          | ❌ No                            |
| Multi-step animation | ❌ No           | ✔️ Yes                           |
| Loops                | ❌ No           | ✔️ Yes                           |
| Runs automatically   | ❌ No           | ✔️ Yes                           |
| Ideal for            | UI interactions | Complex or continuous animations |
| Complexity           | Simple          | Complex                          |

---

# 🎯 Summary

- **Transition = simple, triggered, 2-state animation.**
    
- **Keyframes = complex, timeline-based animation that can run automatically and loop.**
    