---
tags: 
 - css
 - transition
 - delay
---

`transition-delay` defines **how long to wait before a transition starts** after the property value changes.

```css
selector {
  transition-delay: <time>;
}
```

- The value is a **time**, e.g. `2s` (seconds) or `200ms` (milliseconds).
    
- Can be **positive** or **negative**.
    

---

## 🕒 Example

```css
.box {
  width: 100px;
  background: steelblue;
  transition: width 0.5s ease 1s; /* 1s delay */
}

.box:hover {
  width: 200px;
}
```

**What happens:**  
When you hover, the width change waits **1 second** before starting, then animates over **0.5 seconds**.

---

## 🧠 Negative Delay

You can use **negative values** — this means the transition is considered to have **already been running** for that amount of time.

Example:

```css
div {
  transition: transform 2s ease -1s;
}
```

→ The transition starts halfway through (1 second in).  
So it begins in a partially transitioned state and finishes faster.

Useful for staggered or layered effects (like fading items in at slightly different times).

---

## 🎨 Multiple Delays

If you have multiple properties transitioning, you can assign **a separate delay** to each one.

```css
div {
  transition-property: opacity, transform;
  transition-duration: 1s, 1s;
  transition-delay: 0s, 0.3s;
}
```

→ `opacity` starts immediately,  
→ `transform` starts 0.3s later.

---

## 🔄 Use Cases

### 1. **Staggered Animations**

Create a cascading reveal effect:

```css
.item {
  opacity: 0;
  transition: opacity 0.5s ease;
}

.item:nth-child(1) { transition-delay: 0s; }
.item:nth-child(2) { transition-delay: 0.2s; }
.item:nth-child(3) { transition-delay: 0.4s; }

.container:hover .item {
  opacity: 1;
}
```

### 2. **Delayed Hover Effects**

Prevent immediate hover effects (useful for menus or tooltips):

```css
.tooltip {
  opacity: 0;
  transition: opacity 0.2s ease-in 0.4s; /* delay 0.4s */
}

.button:hover + .tooltip {
  opacity: 1;
}
```

### 3. **Sequencing Multiple Transitions**

Animate one property, then another:

```css
div {
  transition: opacity 0.5s ease, transform 0.5s ease 0.5s;
}
```

→ opacity transition starts first,  
→ transform transition starts **after 0.5s**, so they happen one after the other.

---

## 🧭 Summary

|Property|Description|Example|
|---|---|---|
|`transition-delay`|How long to wait before starting the transition|`1s`, `300ms`, `-0.5s`|
|Accepts negative values|Yes — starts “partway through” the transition|`-0.3s`|
|Can list multiple values|Yes, one per property|`0s, 0.2s, 0.4s`|
|Default value|`0s` (no delay)||