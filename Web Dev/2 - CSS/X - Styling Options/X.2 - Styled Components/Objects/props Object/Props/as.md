---
tags: 
 - css
 - styled 
 - components
 - prop
---

### 1. What the `as` Prop Is

- `as` is a **polymorphic prop** provided by styled-components
    
- It allows a styled component to **render as a different HTML element or component**
    
- Styling stays the same; **only the rendered element changes**
    

---

### 2. Basic Example 🧱

```tsx
const Button = styled.button`
  padding: 1rem;
  border-radius: 6px;
`;
```

```tsx
<Button>Button</Button>
<Button as="a" href="/home">Link</Button>
```

Result:

- Same styles
    
- Different underlying elements (`button` vs `a`)
    

---

### 3. Why `as` Exists 🎯

The `as` prop solves:

- **Semantic correctness** (button vs link vs div)
    
- **Accessibility concerns**
    
- **Style reuse without duplication**
    

Without `as`, you would need multiple styled components with identical CSS.

---

### 4. `as` Does NOT Affect Styles 🎨

- CSS defined in the styled component is reused
    
- Selectors like `&:hover` still work
    
- Only the **rendered tag changes**
    

Mental model:

> “`as` changes _what it is_, not _how it looks_.”

---

### 5. Using `as` with Custom Components 🧩

```tsx
const Button = styled.button`
  color: white;
`;

const RouterLink = ({ to, ...props }) => <a href={to} {...props} />;

<Button as={RouterLink} to="/about" />
```

styled-components:

- Forwards props
    
- Applies the generated className to `RouterLink`
    

---

### 6. `as` vs Props-Based Styling ⚖️

|Feature|`as` prop|Regular props|
|---|---|---|
|Changes element type|✅|❌|
|Changes styles|❌|✅|
|Affects semantics|✅|❌|
|Generates new CSS|❌|Often ✅|

---

### 7. `as` vs `forwardedAs` 🚚

```tsx
<Component as="section" />
```

```tsx
<Component forwardedAs="section" />
```

- `as`: changes the **current component**
    
- `forwardedAs`: passes the polymorphic change **down to a child**
    

Used mainly in component libraries.

---

### 8. TypeScript Behavior 🧠

- `as` updates the allowed attributes automatically
    

```tsx
<Button as="a" href="#">OK</Button>   // valid
<Button as="button" href="#">❌</Button>
```

This is one of the strongest advantages of `as` in SC + TS.

---

### 9. Common Use Cases ✅

- Button that sometimes acts as a link
    
- Styled layout component (`Box`) rendered as `section`, `article`, etc.
    
- Design system primitives
    

---

### 10. When NOT to Use `as` ⚠️

Avoid `as` when:

- Behavior differs significantly
    
- Accessibility requirements change (e.g., keyboard handling)
    
- The component becomes semantically misleading
    

In such cases, create a separate component.

---

### 11. Mental Model 🧠

> “`as` is a render-time switch for **element identity**, not a styling mechanism.”