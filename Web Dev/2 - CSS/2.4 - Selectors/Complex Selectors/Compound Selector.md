---
tags:
  - css
  - selector
  - fundamental
---

### 📌 Description

A **compound selector** targets elements that match **all of the simple selectors** in the sequence.

- It’s created by **combining multiple simple selectors** **without spaces**.
    
- The element must meet **all conditions** to be selected.
    
- “Simple selectors” can be:
    
    - **Type selectors** (e.g., `p`)
        
    - **Class selectors** (e.g., `.highlight`)
        
    - **ID selectors** (e.g., `#main`)
        
    - **Attribute selectors** (e.g., `[type="text"]`)
        
    - **Pseudo-classes** (e.g., `:hover`)
        

```ad-note
We can _increase_ specificity by chaining — or “compounding” — selectors together. This way, we give our selector a higher priority when it comes to evaluating two or more matching styles.
```

---

### 📍 Syntax

```css
simpleSelector1simpleSelector2 {
  /* styles */
}
```

- No spaces between them — if there’s a space, it becomes a **descendant selector** instead.
    

---

### 📍 Examples

#### 1. Element + Class

```css
p.note {
  color: blue;
}
```

✅ Targets `<p>` elements **with** the class `note`.

---

#### 2. Element + ID

```css
h1#title {
  font-weight: bold;
}
```

✅ Targets `<h1>` elements **with** the ID `title`.

---

#### 3. Element + Attribute + Pseudo-Class

```css
input[type="email"]:focus {
  border-color: green;
}
```

✅ Targets `<input>` elements **with type `email`** when focused.

---

#### 4. Class + Pseudo-Class

```css
.card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}
```

✅ Targets elements **with** `.card` class when hovered.

---

### 📌 Key Points

- **Compound = AND logic** → all conditions must be true for the element.
    
- Unlike **group selectors** (`,`), compounds do not style multiple independent selectors.
    
- Unlike **descendant selectors** (space), compounds match a **single element**, not parent/child relationships.
    

---

### ⚠️ Best Practices

- Keep them readable — avoid chaining too many selectors.
    
- Prefer **semantic class names** so compounds are meaningful.
    
- Use them for **specific targeting** without overloading HTML with extra IDs.
    
