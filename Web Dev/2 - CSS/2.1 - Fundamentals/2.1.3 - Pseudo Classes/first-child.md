---
tags: 
 - css
 - pseudo
 - class
---

🧭 **What it is**

> `:first-child` is a **structural pseudo-class** that matches an element **only if it is the very first child of its parent**.

---

## 🧪 Basic syntax

```css
:first-child { }
```

Scoped:

```css
li:first-child { }
```

---

## ✅ What it matches

```html
<ul>
  <li>First</li>   <!-- ✅ matches -->
  <li>Second</li>
</ul>
```

```css
li:first-child {
  font-weight: bold;
}
```

---

## ❌ Common confusion (important)

### ❌ “First of this type” ❌

```html
<div>
  <h1>Title</h1>
  <p>Text</p>   <!-- ❌ NOT first-child -->
</div>
```

```css
p:first-child { } /* does NOT match */
```

Because the `<p>` is **not the first child** — `<h1>` is.

---

## ✅ Compare with `:first-of-type`

```css
p:first-of-type { }
```

Matches the **first `<p>` among its siblings**, regardless of position.

---

## 🧩 Text nodes & comments

- Text nodes (including whitespace) **do not affect** `:first-child`
    
- Only **element nodes** are counted
    

✅ Safe to indent HTML.

---

## 🧭 Typical use cases

- Remove margin from the first item
    
- Style the first list item
    
- Add spacing between siblings except the first
    

```css
.card > *:first-child {
  margin-top: 0;
}
```

---

## 🆚 Related selectors

| Selector         | Meaning                    |
| ---------------- | -------------------------- |
| `:first-child`   | First child of parent      |
| `:last-child`    | Last child of parent       |
| `:nth-child(1)`  | Same as `:first-child`     |
| `:first-of-type` | First element of that type |

---

## ⚠️ Pitfall to remember

> `:first-child` cares about **position**, not **tag name**.

If you think “first `<p>`”, you probably want `:first-of-type`.

---

### 🔑 Takeaway

> Use `:first-child` when you truly mean **the very first element inside a parent**, not just the first of a given tag.