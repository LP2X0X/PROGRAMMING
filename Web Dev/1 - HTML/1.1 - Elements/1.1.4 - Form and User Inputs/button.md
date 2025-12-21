---
tags: 
 - html
 - element
 - fundamental
---

## 1. What `<button>` is (semantically)

`<button>` represents a **user-triggered action**.

Use it for:

- Submitting forms
    
- Triggering UI actions (open modal, toggle state)
    
- Any interactive command
    

Do **not** use it for navigation (use `<a>`).

---

## 2. Button types (critical detail)

```html
<button type="button">Action</button>
<button type="submit">Submit</button>
<button type="reset">Reset</button>
```

### Default behavior (important)

- Default `type` = **`submit`**
    
- Inside a `<form>`, clicking without `type` **submits the form**
    

Rule:

> Always specify `type`.

---

## 3. Content model (what can go inside)

`<button>` can contain:

- Text
    
- Inline elements (`span`, `strong`, `svg`)
    
- Even block elements (HTML5)
    

Example:

```html
<button>
  <svg></svg>
  <span>Save</span>
</button>
```

---

## 4. Native behaviors

Buttons provide:

- Keyboard support (`Enter`, `Space`)
    
- Focus management
    
- ARIA role (`role="button"` implicitly)
    
- Disabled state behavior
    

This is why `<button>` is superior to clickable `<div>`s.

---

## 5. Disabled state

```html
<button disabled>Save</button>
```

Behavior:

- Not focusable
    
- Does not fire click events
    
- Communicated to assistive tech
    

CSS:

```css
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## 6. Default UA styles (why buttons feel “weird”)

Browsers apply:

- Platform UI font metrics
    
- Borders
    
- Backgrounds
    
- Padding
    
- Focus rings
    

These styles:

- Differ per OS
    
- Override visual consistency
    

---

## 7. Correct way to reset button styles

Minimal, safe reset:

```css
button {
  font: inherit;
  color: inherit;
  background: none;
  border: none;
  padding: 0;
}
```

Full reset (when fully custom styling):

```css
button {
  all: unset;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
```

⚠️ `all: unset` removes accessibility features — re-add focus styles.

---

## 8. Focus and accessibility (non-negotiable)

Buttons must show focus.

```css
button:focus-visible {
  outline: 2px solid blue;
  outline-offset: 2px;
}
```

Never remove focus without replacement.

---

## 9. Buttons vs links (common mistake)

| Use case    | Correct element |
| ----------- | --------------- |
| Submit form | `<button>`      |
| Toggle UI   | `<button>`      |
| Navigate    | `<a href>`      |

Do not “style a button to act like a link” incorrectly.

---

## 10. Event behavior

Buttons fire:

- `click`
    
- `keydown` (`Enter`, `Space`)
    
- `submit` (if type=submit)
    

You do **not** need JS to handle keyboard activation.

---

## 11. Styling gotchas

### Line-height issues

Buttons often have:

- Larger line-height
    
- Different baselines
    

Fix:

```css
button {
  line-height: inherit;
}
```

### Font issues

Always do:

```css
button {
  font: inherit;
}
```

---

## 12. Button inside forms (important)

```html
<form>
  <button>Submit</button> <!-- submits -->
  <button type="button">Cancel</button>
</form>
```

Explicit `type` prevents bugs.

---

## 13. CSS pseudo-classes you should know

```css
button:hover
button:active
button:focus-visible
button:disabled
```

---

## 14. One-sentence rule to remember

> **Use `<button>` for actions, always set `type`, and reset its font with `font: inherit`.**
