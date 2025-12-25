---
tags: 
 - css
 - property
 - layout
 - shorthand
---

## 🧭 What it is

> **`place-items` is a shorthand alignment property** that sets **both axes** alignment for items **inside a grid or flex container**.

It combines:

- `align-items` (block / cross axis)
    
- `justify-items` (inline / main axis in grid)
    

---

## 🧪 Syntax

```css
place-items: <align-items> <justify-items>;
```

If only one value is provided, it is used for **both**.

```css
place-items: center;
```

Equivalent to:

```css
align-items: center;
justify-items: center;
```

---

## 📦 Where it works

### ✅ Grid containers

Fully supported and commonly used.

```css
.container {
  display: grid;
  place-items: center;
}
```

✔ Centers grid items both horizontally and vertically.

---

### ⚠️ Flex containers (partial)

```css
.container {
  display: flex;
  place-items: center;
}
```

Behavior:

- `align-items` → ✅ applies
    
- `justify-items` → ❌ ignored (Flexbox does not support `justify-items`)
    

So effectively:

```css
align-items: center;
```

---

## 🎯 Common values

|Value|Meaning|
|---|---|
|`start`|Align to start|
|`end`|Align to end|
|`center`|Center|
|`stretch` (default)|Stretch items|
|`baseline`|Align text baselines|

Example:

```css
place-items: start center;
```

---

## 🆚 Related shorthands

| Shorthand       | Expands to                          |
| --------------- | ----------------------------------- |
| `place-items`   | `align-items` + `justify-items`     |
| `place-content` | `align-content` + `justify-content` |
| `place-self`    | `align-self` + `justify-self`       |

---

## 🧠 Important distinctions

- **Items vs content**
    
    - `place-items` → aligns **individual items**
        
    - `place-content` → aligns **the grid tracks as a group**
        
- **Grid vs Flex**
    
    - Grid → both axes work
        
    - Flex → only `align-items` applies
        

---

## ❌ Common misconception

```css
display: flex;
place-items: center;
```

❌ This does **not** center items horizontally and vertically in Flexbox.

Use instead:

```css
display: flex;
align-items: center;
justify-content: center;
```

---

## 🌐 Browser support

- Supported in all modern browsers
    
- Safe for production use
    

---

### 🔑 Takeaway

> **`place-items` is a Grid-first shorthand.**  
> It is excellent for Grid layouts, but only partially effective in Flexbox due to `justify-items` being unsupported there.