---
tags: 
 - css
 - styled
 - component
 - object
 - react
---

### 1. What “props” Are (Important Clarification)

- **Props are a React concept**, not a CSS or JavaScript feature
    
- styled-components **does not create props**
    
- SC **receives props from React components** and uses them to generate styles 🎯
    

---

### 2. How Props Work in styled-components ⚙️

```tsx
const Button = styled.button`
  background: ${({ primary }) => (primary ? "blue" : "gray")};
`;
```

- `primary` comes from React:
    
    ```tsx
    <Button primary />
    ```
    
- React passes props → SC reads them → SC generates CSS
    

---

### 3. Props Are Read-Only in SC 🔒

- styled-components **cannot change props**
    
- It only **consumes props to compute styles**
    
- Props flow direction:
    
    **Parent → Component → styled-components**
    

---

### 4. Props Are Available Everywhere in Styles 🧠

Props can be used in:

- Property values
    
- Conditional blocks
    
- Nested selectors
    
- Media queries
    

```ts
color: ${({ theme, disabled }) =>
  disabled ? theme.gray : theme.primary};
```

---

### 5. Why `$`-Prefixed Props Exist 

```ts
const Button = styled.button<{ $primary?: boolean }>`
  background: ${({ $primary }) => ($primary ? "blue" : "gray")};
`;
```

Reason:

- Prevents invalid HTML attributes like:
    
    ```html
    <button primary="true">
    ```
    
- `$primary` is **used only for styling**
    
- It is **not forwarded to the DOM**
    

Best practice ✔️

---

### 6. Props vs Theme 🎨

| Aspect   | Props           | Theme             |
| -------- | --------------- | ----------------- |
| Scope    | Per component   | Global            |
| Source   | JSX attributes  | `<ThemeProvider>` |
| Use case | Variants, state | Design tokens     |

Both are accessed the same way:

```ts
${({ theme, size }) => ...}
```

---

### 7. Props Enable Conditional CSS 🧪

```ts
${({ $active }) =>
  $active &&
  css`
    border: 2px solid blue;
  `}
```

- `false`, `null`, `undefined` → ignored by SC
    
- Enables clean conditional styling
    

---

### 8. Props Are Evaluated at Render Time 🔄

- When props change → component re-renders
    
- SC recalculates styles
    
- May generate a **new class name**
    

This is why excessive prop-based styling can affect performance ⚠️

---

### 9. Props Do NOT Exist in Vanilla JS ❌

- No React → no components → no props
    
- styled-components **depends on a component framework**
    
- Vanilla JS alternatives:
    
    - Classes
        
    - Data attributes
        
    - CSS variables
        

---

### 10. Mental Model 🧠

> “styled-components is a function that turns **React props + CSS** into a **generated class name**.”

---

### 11. When to Use Props in SC ✅

Use props for:

- Variants (`primary`, `secondary`)
    
- UI state (`active`, `disabled`)
    
- Size differences (`small`, `large`)
    

Avoid props for:

- Purely static styles
    
- Global design decisions (use theme instead)
    