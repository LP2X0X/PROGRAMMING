---
tags: 
 - styled
 - components
 - global
 - style
---

In **styled-components**, you _do not technically need_ to create a global style component — but doing so aligns global CSS with the same **styling model, guarantees, and tooling** as the rest of your app.

---

## 1. styled-components is **CSS-in-JS**, not CSS loading

A traditional global CSS file is:

- Loaded by the browser
    
- Parsed once
    
- Applied outside React’s lifecycle
    

`createGlobalStyle` is:

- **Generated and injected at runtime**
    
- Scoped to the React app lifecycle
    
- Controlled by JavaScript and React
    

This keeps **one styling paradigm** instead of mixing two.

---

## 2. Consistent access to **JS features**

Global styles often need:

- Theme values
    
- Design tokens
    
- Environment-based logic
    

With `createGlobalStyle` you can do:

```js
const GlobalStyle = createGlobalStyle`
  body {
    font-family: ${({ theme }) => theme.fonts.main};
    background: ${({ theme }) => theme.colors.bg};
  }
`;
```

A `.css` file **cannot**:

- Read theme context
    
- React to props
    
- Change per environment at runtime
    

---

## 3. Predictable ordering & overrides

styled-components controls:

- Injection order
    
- Specificity
    
- Deduplication
    

This avoids:

- CSS cascade surprises
    
- Accidental overrides from import order
    
- Conflicts between global CSS and component styles
    

Everything is inserted **exactly when React renders it**.

---

## 4. Easier co-location & maintainability

Global styles become:

- Part of the component tree
    
- Explicitly mounted (e.g., at `<App />`)
    
- Easier to reason about and remove
    

```jsx
function App() {
  return (
    <>
      <GlobalStyle />
      <Layout />
    </>
  );
}
```

No hidden side effects from “some CSS file imported somewhere”.

---

## 5. Works better with SSR and code-splitting

styled-components can:

- Extract critical CSS on the server
    
- Avoid flashing unstyled content
    
- Split styles with routes
    

A raw global CSS file is **always fully loaded**, even when unused.

---

## 6. When a global CSS file _is_ still acceptable

Using a global CSS file is reasonable for:

- CSS resets (normalize.css)
    
- Third-party libraries
    
- Legacy migrations
    

styled-components does **not forbid** global CSS — it **recommends consistency**.

---

## Bottom line

You create a “global style component” because:

- styled-components treats **styles as components**
    
- Global CSS becomes **explicit, theme-aware, and lifecycle-bound**
    
- You avoid mixing styling systems
    

It is not a requirement — it is an **architectural choice** that improves predictability in React applications.