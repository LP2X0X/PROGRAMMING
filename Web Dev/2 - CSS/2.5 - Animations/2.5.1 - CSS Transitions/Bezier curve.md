---
tags: 
 - css
 - transition
 - math
---

### 🧩 Syntax

```
cubic-bezier(x1, y1, x2, y2)
```

- The curve **always starts at (0,0)** and **ends at (1,1)**
    
- `(x1, y1)` and `(x2, y2)` are **control points**
    
- `x` = time progress (0 → 1)
    
- `y` = animation progress (0 → 1)
    

```ad-note
For intermediate points, the values of `x` must be in the interval `0..1`, `y` can be anything.
```

---

### 📈 How It Works

|Position|Meaning|
|---|---|
|🔹 `x1, x2`|The time: `0` – the start, `1` – the end of `transition-duration`.|
|🔹 `y1, y2`|The completion of the process: `0` – the starting value of the property, `1` – the final value.|
|🕐 Curve above diagonal|Fast at start, slows near end|
|🐢 Curve below diagonal|Slow start, speeds up later|

---

### ⚙️ Built-in Equivalents

|Name|Equivalent Bézier|Description|
|---|---|---|
|`linear`|`cubic-bezier(0, 0, 1, 1)`|Constant speed|
|`ease`|`cubic-bezier(0.25, 0.1, 0.25, 1.0)`|Gentle, default|
|`ease-in`|`cubic-bezier(0.42, 0, 1.0, 1.0)`|Starts slow|
|`ease-out`|`cubic-bezier(0, 0, 0.58, 1.0)`|Ends slow|
|`ease-in-out`|`cubic-bezier(0.42, 0, 0.58, 1.0)`|Slow start & end|

---

### 💡 Common Custom Curves

|Effect|Bézier|Use case|
|---|---|---|
|⚡ Snappy|`cubic-bezier(0.33, 1.2, 0.68, 1)`|Button pop or fast snap|
|🧘‍♀️ Soft pop|`cubic-bezier(0.2, 0.9, 0.2, 1)`|Natural growth effect|
|🚀 Fast finish|`cubic-bezier(0.1, 0.7, 0.4, 1)`|Panels, reveals|
|🎯 Sharp start|`cubic-bezier(0.7, 0, 1, 0.2)`|Quick in, immediate feel|
|🎈 Bouncy|`cubic-bezier(0.5, 1.6, 0.2, 1)`|Playful hover bounce|

---

### 🧱 Example

```css
.element {
  transition: transform 350ms cubic-bezier(0.2, 0.9, 0.2, 1);
}

.element:hover {
  transform: translateY(-6px) scale(1.05);
}
```

---

### Bezier curve can make the animation exceed its range

The control points on the curve can have any `y` coordinates: even negative or huge ones. Then the Bezier curve would also extend very low or high, making the animation go beyond its normal range.

In the example below the animation code is:

```css
.train {
  left: 100px;
  transition: left 5s cubic-bezier(.5, -1, .5, 2);
  /* click on a train sets left to 450px */
}
```

The property `left` should animate from `100px` to `400px`.

But if you click the train, you’ll see that:

- First, the train goes _back_: `left` becomes less than `100px`.
- Then it goes forward, a little bit farther than `400px`.
- And then back again – to `400px`.

---

### 🪄 Tips

- 🕒 **Duration:** 150–600 ms (shorter for micro-interactions)
    
- 🔁 **Reuse curves:** Define a few presets (`--ease-snappy`, `--ease-soft`)
    
- ⚠️ **Overshoot (y > 1)** is fun but easy to overdo
    
- 👀 **Visualize:** use tools like [cubic-bezier.com](https://cubic-bezier.com)
    

---