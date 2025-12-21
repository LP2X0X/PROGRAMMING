---
tags: 
 - html
 - attribute
---

## 1. What the `target` Attribute Is

The `target` attribute specifies **where the linked document or form response should be opened**.

It is most commonly used with:

- `<a>`
    
- `<form>`
    
- `<area>`
    

---

## 2. Common Values of `target`

### `_self` (default)

Opens the link in the **same browsing context** (same tab/frame).

```html
<a href="/home" target="_self">Home</a>
```

---

### `_blank`

Opens the link in a **new tab or window**.

```html
<a href="https://example.com" target="_blank">External</a>
```

⚠ **Security note**  
Always pair with:

```html
rel="noopener noreferrer"
```

To prevent tab-nabbing attacks.

---

### `_parent`

Opens the link in the **parent frame**.

Used mainly with iframes.

```html
<a href="/page" target="_parent">
```

---

### `_top`

Opens the link in the **top-level browsing context**, breaking out of all frames.

```html
<a href="/page" target="_top">
```

---

### Named target

Opens the link in a **named browsing context** (window or iframe).

```html
<iframe name="contentFrame"></iframe>

<a href="/help" target="contentFrame">Help</a>
```

If a window or iframe with that name exists, it is reused.

---

## 3. `target` with `<form>`

Controls where the form submission result is displayed.

```html
<form action="/submit" target="_blank">
```

Useful for:

- Previewing submissions
    
- Submitting without replacing current page
    

---

## 4. Default Behavior

- If `target` is omitted → `_self`
    
- Each browsing context (tab, iframe) has a **name**
    
- `_blank` always creates a new context
    

---

## 5. Security & Best Practices

### Use with `_blank`

```html
<a href="..." target="_blank" rel="noopener noreferrer">
```

Why:

- Prevents access to `window.opener`
    
- Avoids performance and security risks
    

---

## 6. When NOT to use `target="_blank"`

- Internal navigation
    
- SPA routing (React Router, etc.)
    
- When it breaks expected user flow
    

In SPAs, use routing instead of opening new tabs.

---

## 7. Summary

- `target` defines **where navigation opens**
    
- `_self` is default
    
- `_blank` opens a new tab (use `rel="noopener"`)
    
- `_parent` and `_top` are mainly for frames
    
- Named targets allow iframe/window reuse
    