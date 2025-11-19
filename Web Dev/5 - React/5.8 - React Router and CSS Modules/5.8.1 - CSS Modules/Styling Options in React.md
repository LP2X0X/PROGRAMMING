---
tags: 
 - react
 - note
 - style
 - css
---

![[Pasted image 20250925143810.png|center|700]]

### 1. **Inline CSS**

- **Where?** Inside JSX elements.
    
- **How?** Using the `style` prop, which accepts a JavaScript object.
    
- **Scope:** Local → affects only that specific element.
    
- **Based on:** CSS.
    
- ✅ **Pros:** Quick for small tweaks, no external files, scoped to one element.
    
- ❌ **Cons:** No pseudo-classes (`:hover`, `:focus`) or media queries directly, styles can get messy if overused.
    
- **Example:**
    
    ```jsx
    <h1 style={{ color: "red", fontSize: "20px" }}>Hello</h1>
    ```
    

---

### 2. **CSS or Sass file**

- **Where?** External `.css` or `.scss` file, imported into React.
    
- **How?** Apply via `className` prop.
    
- **Scope:** Global → styles can leak and override each other (CSS cascade).
    
- **Based on:** CSS (with optional Sass features like variables, nesting).
    
- ✅ **Pros:** Familiar, simple, works well for small apps. Sass adds power (variables, mixins).
    
- ❌ **Cons:** Hard to scale in large apps → naming conflicts, hard maintenance.
    
- **Example:**
    
    ```jsx
    import "./App.css";
    
    function App() {
      return <h1 className="title">Hello</h1>;
    }
    /* App.css */
    .title { color: blue; }
    ```
    

---

### 3. **CSS Modules**

- **Where?** One external CSS file per component, named with `.module.css`.
    
- **How?** Import CSS as an object, then apply classes with `className`.
    
- **Scope:** Component → styles are automatically scoped (unique class names).
    
- **Based on:** CSS.
    
- ✅ **Pros:** No global style collisions, good for medium to large apps.
    
- ❌ **Cons:** More boilerplate than plain CSS, still separate from JS logic.
    
- **Example:**
    
    ```jsx
    import styles from "./Button.module.css";
    
    function Button() {
      return <button className={styles.primary}>Click</button>;
    }
    /* Button.module.css */
    .primary { background: green; color: white; }
    ```
    

---

### 4. **CSS-in-JS (e.g., styled-components, emotion)**

- **Where?** In JS/TS files or external styled files.
    
- **How?** Create styled components directly in JavaScript.
    
- **Scope:** Component → styles are scoped to the component.
    
- **Based on:** JavaScript (styles written as JS).
    
- ✅ **Pros:** Co-locates styles with components, dynamic styling (props, themes), avoids naming collisions.
    
- ❌ **Cons:** Bigger JS bundle size, runtime overhead, sometimes slower performance.
    
- **Example (styled-components):**
    
    ```jsx
    import styled from "styled-components";
    
    const Button = styled.button`
      background: blue;
      color: white;
      padding: 10px;
    `;
    
    function App() {
      return <Button>Click</Button>;
    }
    ```
    

---

### 5. **Utility-first CSS (e.g., TailwindCSS)**

- **Where?** JSX elements directly.
    
- **How?** Use prebuilt utility classes via `className`.
    
- **Scope:** Local → applies only to that element.
    
- **Based on:** CSS (utility class framework).
    
- ✅ **Pros:** Very fast development, consistent design system, avoids writing custom CSS.
    
- ❌ **Cons:** Class-heavy JSX (harder to read), learning curve for utilities.
    
- **Example (TailwindCSS):**
    
    ```jsx
    function App() {
      return <h1 className="text-2xl font-bold text-blue-500">Hello</h1>;
    }
    ```
    

---

### 6. **UI Libraries (alternative to custom CSS)**

- Examples: **Material UI (MUI), Chakra UI, Mantine, Ant Design, Bootstrap React**.
    
- These provide **ready-to-use components** (buttons, cards, modals) with built-in styling and themes.
    
- ✅ **Pros:** Speeds up development, consistent UI, responsive out of the box.
    
- ❌ **Cons:** Less flexibility (harder to customize deeply), larger bundle size.
    
- **Example (Chakra UI):**
    
    ```jsx
    import { Button } from "@chakra-ui/react";
    
    function App() {
      return <Button colorScheme="teal">Click Me</Button>;
    }
    ```
    

---

## 🔑 Summary

- **Inline CSS** → Quick, local, but limited.
    
- **CSS/Sass file** → Global, easy, but can cause conflicts.
    
- **CSS Modules** → Scoped per component, safer for bigger apps.
    
- **CSS-in-JS** → Co-locate styles with logic, dynamic, but heavier.
    
- **Utility-first (Tailwind)** → Fast and consistent, but verbose classnames.
    
- **UI Libraries** → Prebuilt solutions, fastest to build apps, but less flexible.
    