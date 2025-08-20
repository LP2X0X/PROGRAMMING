---
tags:
  - css
  - value
  - fundamental
---

The **`inherit`** value in CSS is a special keyword you can use to make an element explicitly take a property’s computed value from its parent element, even if that property does not normally inherit by default.

---

### 🔹 How It Works

- Normally, **only certain CSS properties** (like `color`, `font-family`, `line-height`) inherit automatically from their parent element.
    
- Other properties (like `border`, `margin`, `padding`, `background`) do **not** inherit unless you explicitly tell them to.
    
- By setting a property to `inherit`, you are telling the browser:
    
    > “Whatever computed value my parent has for this property, I want exactly the same.”
    

---

### 🔹 Example

```css
div {
  border: 2px solid red;
}

p {
  border: inherit; /* Forces paragraph to use the same border as its parent div */
}
```

If a `<p>` is inside the `<div>`, it will now have the same 2px solid red border, even though `border` does not normally inherit.

---

### 🔹 When to Use `inherit`

- **Consistency in style** – for example, forcing all text inside a container to share the same `color` and `font-family`.
    
- **Overriding browser defaults** – sometimes browsers give elements default styles that break your design, and `inherit` can fix that.
    
- **Theming** – when you want child elements to automatically follow the styling of a themed container.
    

---

### 🔹 Tips & Best Practices

✅ Use `inherit` sparingly—too much can make debugging harder.  
✅ Great for typography (fonts, colors) to ensure consistent look.  
✅ Use in resets or utility classes when you want children to strictly follow their parent’s styling.  
❌ Avoid using `inherit` for layout properties like `width`, `margin`, `padding` unless you really mean it, as it can create unexpected results.