---
tags: 
 - react
 - prop
 - technique
---

A React component can use a prop named `type` to determine and apply class names, making the component reusable.

- **Prop name:** `type`
    
- **Purpose:** Determines the styling or variant of the component.
    
- **Example types:** `"primary" | "secondary" | "danger"`
    

**TypeScript example:**

```js
import { Link } from 'react-router-dom';

function Button({ to, type, disabled, children }) {
  const base = `inline-block rounded-full bg-yellow-400 font-semibold tracking-wide text-stone-800 uppercase transition-colors duration-300 hover:bg-yellow-300 focus:ring focus:ring-yellow-300 focus:ring-offset-2 focus:outline-none `;

  const styles = {
    primary: base + `px-4 py-3 sm:px-6 sm:py-4`,
    small: base + `px-4 py-2 md:px-5 md:py-2.5`,
  };

  if (to) {
    return (
      <Link to={to} className={styles[type]}>
        {children}
      </Link>
    );
  }
  return (
    <button className={styles[type]} disabled={disabled}>
      {children}
    </button>
  );
}

export default Button;
```

```ad-tip
You can create an object here to map the styles you wanted with the "type".
It should have a base style which will be common to all the component instances.
```

```ad-note
Be careful, since the value is a string — no space between base string and others can prevent the style from being applied correctly.
```

**Usage:**

```tsx
<Button type="primary">Click Me</Button>
<Button type="danger">Delete</Button>
```

✅ Just the single prop `type` is enough to pass the class name.