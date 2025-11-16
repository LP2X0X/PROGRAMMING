---
tags: 
 - css
 - keyword
 - value
 - fundamental
---

`unset` tells CSS:

> **“If this property normally inherits → use the parent’s value.  
> Otherwise → use the initial (spec default) value.”**

It's **smart** because it adapts to the property.

---

## 🧠 **How `unset` decides what to do**

### **1️⃣ For _inherited_ properties (e.g., color, font-size)**

`unset` → behaves like **inherit**

Example:

```css
color: unset;  
/* same as color: inherit */
```

### **2️⃣ For _non-inherited_ properties (e.g., margin, display)**

`unset` → behaves like **initial**

Example:

```css
margin: unset;
/* same as margin: initial → becomes 0 */
```

---

## 📌 **Summary Table**

|Property type|Real effect of `unset`|Example|
|---|---|---|
|**Inherited**|`inherit`|color, font-size, line-height|
|**Not inherited**|`initial`|margin, padding, display, position|

---

## 💡 Why `unset` is useful

### ✔ **Makes resets easier when you’re not sure if a property inherits**

You don’t need to remember which props inherit — CSS decides for you.

### ✔ **Great in component libraries**

Useful for resetting unknown 3rd-party styles.

### ✔ **Safer than `all: initial`**

Because it doesn’t override inherited text styling unless it’s meant to.

### ✔ **Nice for overriding user-agent styles**

Example: removing `<a>` default blue/underline.

---

## 🧨 Example: Resetting link styles

```css
a {
  all: unset;
  cursor: pointer;
  color: inherit;  /* keep parent color */
}
```

But if you only want to remove certain styles:

```css
a {
  text-decoration: unset; /* becomes none */
  color: unset; /* inherits parent color */
}
```

---

## 🎯 When to use `unset` vs `initial` vs `inherit`

|Keyword|Best use case|
|---|---|
|**unset**|General resets when you’re unsure about inheritance|
|**initial**|Hard reset to spec default (ignore parent styles)|
|**inherit**|Force consistency across a group of components|
|**revert**|Undo your custom CSS, returning to UA, browser or user stylesheet|

---

## 🔥 Quick Mental Model

- **initial → spec default**
    
- **inherit → parent’s value**
    
- **unset → auto choose between initial or inherit**
    
- **revert → roll back your CSS and frameworks**
    