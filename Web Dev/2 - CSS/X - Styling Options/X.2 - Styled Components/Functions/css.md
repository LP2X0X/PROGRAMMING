---
tags: 
 - css
 - styled
 - component
 - function
---

### Definition

The `css` function is a **helper utility** provided by styled-components that allows you to **define reusable blocks of CSS**. These blocks can be interpolated into styled components or conditionally applied based on props.

It does **not** create a component. Instead, it returns a **CSS fragment** that styled-components knows how to merge into a component’s styles.

---

### Purpose

The `css` function exists to solve three main problems:

1. **Style reuse** without duplicating CSS
    
2. **Conditional styling** based on props
    
3. **Better type inference** (especially in TypeScript)
    

---

### Basic Usage

```ts
import { css } from "styled-components";

const hoverStyle = css`
  &:hover {
    background-color: var(--color-brand-700);
  }
`;
```

This `hoverStyle` is **not rendered by itself**. It must be interpolated into a styled component.

---

### Using `css` Inside a Styled Component

```ts
const Button = styled.button`
  padding: 1rem;
  background: var(--color-brand-500);

  ${hoverStyle}
`;
```

The CSS block is **inlined** into the final generated stylesheet.

---

### Conditional Styling with Props

```ts
const primaryStyle = css`
  background-color: blue;
  color: white;
`;

const Button = styled.button<{ $primary?: boolean }>`
  padding: 1rem;

  ${({ $primary }) => $primary && primaryStyle}
`;
```

Key point:

- The expression returns **either a CSS block or `false`**
    
- styled-components ignores `false`, `null`, and `undefined`
    

---

### Why `css` Is Needed for Conditions

This **does not work correctly**:

```ts
${props => props.$primary && `
  background: blue;
`}
```

Reason:

- Plain template strings are treated as **raw strings**
    
- TypeScript loses type information
    
- Nested selectors and interpolations may break
    

The `css` function:

- Preserves **structure**
    
- Preserves **nesting**
    
- Preserves **types**
    

---

### Reusing Styles Across Components

```ts
const cardBase = css`
  padding: 2rem;
  border-radius: 8px;
`;

const Card = styled.div`
  ${cardBase}
`;

const Modal = styled.section`
  ${cardBase}
`;
```

This is similar to a **mixin** in traditional CSS preprocessors.

---

### Using `css` with Media Queries

```ts
const mobileOnly = css`
  @media (max-width: 40em) {
    display: none;
  }
`;

const Sidebar = styled.aside`
  ${mobileOnly}
`;
```

---

### Interaction with `&` Selector

Inside a `css` block, `&` still refers to the **final component selector**:

```ts
const hover = css`
  &:hover {
    opacity: 0.8;
  }
`;
```

When interpolated, `&` resolves to the component class.

---

### Return Value (Important Concept)

```ts
css`...`
```

Returns:

- A **Styled Components CSS object**
    
- Not a string
    
- Not a component
    

This is why it can be composed, nested, and safely interpolated.

---

### Mental Model

Think of `css` as:

> “A typed, structured CSS fragment that can be conditionally or repeatedly injected into components.”

---

### When to Use `css`

Use `css` when you need:

- Reusable style logic
    
- Conditional blocks of styles
    
- Clean separation of styling concerns
    
- Strong TypeScript support
    

Do **not** use it for:

- One-off static styles
    
- Styles that are never reused or conditional
    

---

If you want, I can next:

- Compare `css` vs mixins vs plain interpolation
    
- Explain how `css` affects generated class names
    
- Show best practices specific to React + TypeScript usage