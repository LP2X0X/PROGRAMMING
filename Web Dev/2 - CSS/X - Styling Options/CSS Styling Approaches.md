---
tags: 
 - css
 - styling 
 - approach
---

## 1️⃣ Vanilla CSS (Native CSS)

**What it is**

- Plain CSS interpreted directly by the browser
    
- No build step required
    

**Examples**

```css
.button {
  color: red;
}
```

**Includes**

- CSS variables (`--color`)
    
- Media queries
    
- Modern selectors (`:has()`, `:is()`)
    
- Native nesting (where supported)
    

**Pros**

- Zero tooling
    
- Best performance
    
- Full browser support
    
- Standards-based
    

**Cons**

- No logic (loops, conditionals)
    
- Manual organization
    

---

## 2️⃣ CSS Preprocessors

**What they are**

- Superset languages that compile to CSS
    
- Add syntax sugar and limited logic
    

### Common preprocessors

- **Sass / SCSS** (most popular)
    
- Less
    
- Stylus
    

**Features**

- Variables
    
- Nesting
    
- Mixins
    
- Functions
    
- Partials & imports
    
- Loops and conditionals
    

```scss
$primary: blue;

.button {
  color: $primary;
}
```

**Pros**

- Better organization
    
- Less repetition
    

**Cons**

- Build step required
    
- Features now overlap with native CSS
    

---

## 3️⃣ PostCSS (CSS Transformers)

**What it is**

- A tool that **transforms CSS via plugins**
    
- Not a language by itself
    

**Common uses**

- Autoprefixing
    
- Polyfilling modern CSS
    
- Minification
    
- Nesting (plugin-based)
    

```css
/* Input */
.button {
  color: red;
}
```

```css
/* Output */
.button {
  color: red;
  -webkit-color: red;
}
```

**Pros**

- Highly modular
    
- Works with vanilla CSS
    

**Cons**

- Tooling complexity
    
- Plugin-dependent behavior
    

---

## 4️⃣ CSS Modules (Scoped CSS)

**What it is**

- Vanilla CSS with **local scoping**
    
- Class names are hashed at build time
    

```css
/* Button.module.css */
.button {
  color: red;
}
```

```js
import styles from './Button.module.css';
```

**Pros**

- No runtime cost
    
- Prevents class collisions
    
- Still “real CSS”
    

**Cons**

- Build tooling required
    
- Limited dynamic styling
    

---

## 5️⃣ CSS-in-JS (Runtime Styling)

**What it is**

- CSS written inside JavaScript
    
- Styles generated at runtime
    

### Examples

- styled-components
    
- Emotion
    
- Stitches
    

```js
const Button = styled.button`
  color: red;
`;
```

**Pros**

- Dynamic styles
    
- Co-located logic
    
- Theming support
    

**Cons**

- Runtime overhead
    
- JS dependency
    
- Harder static analysis
    

---

## 6️⃣ Utility-First CSS

**What it is**

- Styling via predefined utility classes
    
- Minimal custom CSS
    

### Example

```html
<button class="bg-blue-500 text-white p-4">
```

**Popular frameworks**

- Tailwind CSS
    
- Windi CSS
    
- UnoCSS
    

**Pros**

- Fast development
    
- No naming decisions
    
- Highly consistent
    

**Cons**

- HTML verbosity
    
- Learning curve
    
- Less semantic markup
    

---

## 7️⃣ Design Systems & Component Libraries

**What they are**

- Opinionated styling + components
    
- Often built on top of other approaches
    

### Examples

- Material UI
    
- Chakra UI
    
- Ant Design
    

**Pros**

- Rapid development
    
- Consistency
    
- Accessibility baked in
    

**Cons**

- Less control
    
- Bundle size
    
- Opinionated design
    

---

## 8️⃣ Native Web Components Styling

**What it is**

- Styling custom elements via:
    
    - Shadow DOM
        
    - CSS custom properties
        

```js
this.attachShadow({ mode: 'open' });
```

**Pros**

- True encapsulation
    
- Framework-agnostic
    

**Cons**

- More complex mental model
    
- Limited tooling
    

---

## 🧠 Mental hierarchy (from low → high abstraction)

```
Vanilla CSS
  → Preprocessors
    → PostCSS
      → CSS Modules
        → CSS-in-JS
          → Utility CSS
            → Component Libraries
```

---

## ✅ Key takeaway

> All modern styling approaches **compile down to CSS**.  
> The difference is **where, when, and how** that CSS is generated and scoped.