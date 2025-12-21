---
tags: 
 - html
 - attribute
---

🧭 **What it does**

> `autocomplete` controls whether—and **how**—the browser can automatically fill in form fields using saved user data.

It improves:

- ⏱️ Speed
    
- ♿ Accessibility
    
- 📱 Mobile keyboard behavior
    

---

🧩 **Where it’s used**  
✅ On:

- `<form>`
    
- `<input>`
    
- `<textarea>`
    
- `<select>`
    

---

🧪 **Basic syntax**

```html
<input autocomplete="value">
```

---

## 🎛️ Top-level control

### 🔄 `on` (default)

```html
<form autocomplete="on">
```

📝 Browser decides what to autofill.

### 🚫 `off`

```html
<form autocomplete="off">
```

📝 Requests browser **not** to autofill  
⚠️ Browsers may still ignore this for login fields.

---

## 🧠 Autofill tokens (important)

Use **semantic tokens** so browsers understand _what the field represents_.

---

### 👤 Personal info

- `name`
    
- `given-name`
    
- `family-name`
    
- `nickname`
    
- `username`
    
- `email`
    
- `tel`
    
- `photo`
    

```html
<input name="email" autocomplete="email">
```

---

### 🏠 Address

- `street-address`
    
- `address-line1`
    
- `address-line2`
    
- `address-level1` (state/province)
    
- `address-level2` (city)
    
- `postal-code`
    
- `country`
    
- `country-name`
    

```html
<input autocomplete="postal-code">
```

---

### 💳 Payment

- `cc-name`
    
- `cc-number`
    
- `cc-exp`
    
- `cc-exp-month`
    
- `cc-exp-year`
    
- `cc-csc`
    

⚠️ Use carefully and only over HTTPS.

---

### 🔐 Authentication

- `current-password`
    
- `new-password`
    
- `one-time-code`
    

```html
<input type="password" autocomplete="current-password">
```

---

### 🧭 Special modifiers

#### 🧑‍🤝‍🧑 Sectioning

```html
<input autocomplete="section-shipping address-line1">
```

📝 Separates multiple addresses in one form.

#### 🏷️ Context qualifiers

- `home`
    
- `work`
    
- `mobile`
    
- `fax`
    
- `pager`
    

```html
<input autocomplete="tel-mobile">
```

---

## ❌ What NOT to do

```html
<input autocomplete="false">   <!-- invalid -->
<input autocomplete="no">      <!-- invalid -->
```

Only valid values are **defined tokens**, `on`, or `off`.

---

## ♿ Accessibility & UX notes

- Improves screen reader context
    
- Enables better mobile keyboards (email, number, tel)
    
- Reduces user input errors
    

---

## 🧭 Best practices

✅ Always pair with:

- `name`
    
- correct `type`
    
- proper `label`
    

❌ Avoid disabling autocomplete unless absolutely necessary.

---

## 🌐 Browser behavior notes

- Browsers may override `off`
    
- Password managers use `autocomplete` heavily
    
- Token accuracy > browser heuristics
    

---

### 🔑 Quick takeaway

> `autocomplete` is not just on/off — it’s a **semantic hint system** that helps browsers fill forms correctly, faster, and more accessibly.