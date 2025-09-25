---
tags: 
 - css
 - modules
 - composing
 - class
---

In **CSS Modules**, you can **compose** classes to reuse styles across multiple selectors without repeating code. This is useful when you want to build on top of an existing class or share utility styles.

### Syntax

```css
/* button.module.css */
.base {
  padding: 10px;
  border-radius: 4px;
  font-weight: bold;
}

.primary {
  composes: base;
  background: blue;
  color: white;
}

.secondary {
  composes: base;
  background: gray;
  color: black;
}
```

### How it works

- `composes: base;` means that the `.primary` and `.secondary` classes will **include all styles** from `.base`.
    
- When imported into your component, the resulting class names get merged into a unique scoped class string.
    

### Usage in React

```jsx
import styles from './button.module.css';

function App() {
  return (
    <>
      <button className={styles.primary}>Primary</button>
      <button className={styles.secondary}>Secondary</button>
    </>
  );
}
```

### Notes

- You can compose from multiple classes:
    
    ```css
    bigPrimary {
      composes: base primary;
      font-size: 20px;
    }
    ```
    
- You can compose from **other files**:
    
    ```css
    /* card.module.css */
    .shadow {
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }
    
    /* button.module.css */
    .fancy {
      composes: shadow from './card.module.css';
      border: 2px solid red;
    }
    ```
    
- `composes` only merges **class names**, not selectors like `:hover` or `@media` rules—you need to duplicate those separately.
    