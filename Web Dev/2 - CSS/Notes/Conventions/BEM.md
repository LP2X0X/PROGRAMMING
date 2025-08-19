---
tags:
  - css
  - note
  - convention
---

## 💡 What is BEM?

**BEM** stands for:

- **B**: Block
    
- **E**: Element
    
- **M**: Modifier
    

It’s a naming convention for writing **clean, reusable, and readable** CSS — especially helpful in large or complex projects.

---

## ✅ BEM Naming Structure

### 1. **Block**

A standalone, reusable component.

```html
<div class="menu"></div>
```

- Example: `menu`, `button`, `card`, `product`
    

---

### 2. **Element**

A part of the block that **depends on the block**.

```html
<div class="menu">
  <div class="menu__item"></div>
</div>
```

- Use `__` to connect the element to the block.
    
- Example: `menu__item`, `card__title`, `product__price`
    

---

### 3. **Modifier**

A **variation** or **state** of a block or element.

```html
<button class="button button--large"></button>
```

- Use `--` to define a modifier.
    
- Example: `button--large`, `menu__item--selected`, `card--highlighted`
    

---

### 🔧 Combined Example:

```html
<article class="product-card product-card--featured">
  <h2 class="product-card__title">Perfume</h2>
  <p class="product-card__description">A floral scent...</p>
  <button class="product-card__button product-card__button--primary">
    Add to Cart
  </button>
</article>
```

Here’s what each part means:

|Class|Meaning|
|---|---|
|`product-card`|Block: the main container|
|`product-card--featured`|Modifier: a special version of the card|
|`product-card__title`|Element: the title inside the card|
|`product-card__button--primary`|Modifier: a variation of the button|

---

## 📦 Why use BEM?

- ✅ Makes CSS **modular and reusable**
    
- ✅ Avoids class name conflicts
    
- ✅ Makes your HTML & CSS easier to **read and maintain**
    
- ✅ Great for **large teams or projects**
    

---

### ✅ Summary:

|Part|Syntax|Example|
|---|---|---|
|Block|`block`|`card`, `menu`|
|Element|`block__element`|`card__title`|
|Modifier|`block--modifier` or `block__element--modifier`|`card--featured`, `card__title--large`|