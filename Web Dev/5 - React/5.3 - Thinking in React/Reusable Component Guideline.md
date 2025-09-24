---
tags: 
 - react
 - reusable
 - component
---

## 1. Identify the UI Piece You Want to Reuse

Look for repeating patterns in your UI.  
Example: A **Button** that appears in multiple places with slight differences (color, size, text).

---

## 2. Create a Component File

Inside `src/components/`, create a new file:

```bash
src/components/Button.jsx
```

---

## 3. Define a Functional Component

Start simple:

```jsx
// Button.jsx
export default function Button({ label }) {
  return <button>{label}</button>;
}
```

Usage:

```jsx
<Button label="Click Me" />
<Button label="Submit" />
```

---

## 4. Add Props for Flexibility

Add props to control styles and behavior:

```jsx
// Button.jsx
export default function Button({ label, onClick, type = "primary" }) {
  const baseStyle = "px-4 py-2 rounded font-medium";
  const typeStyle =
    type === "primary"
      ? "bg-blue-500 text-white hover:bg-blue-600"
      : "bg-gray-200 text-black hover:bg-gray-300";

  return (
    <button className={`${baseStyle} ${typeStyle}`} onClick={onClick}>
      {label}
    </button>
  );
}
```

Usage:

```jsx
<Button label="Save" type="primary" onClick={() => alert("Saved")} />
<Button label="Cancel" type="secondary" onClick={() => alert("Canceled")} />
```

---

## 5. Allow Children (More Flexible Than `label`)

Sometimes you want icons or custom text:

```jsx
// Button.jsx
export default function Button({ children, onClick, type = "primary" }) {
  const baseStyle = "px-4 py-2 rounded font-medium";
  const typeStyle =
    type === "primary"
      ? "bg-blue-500 text-white hover:bg-blue-600"
      : "bg-gray-200 text-black hover:bg-gray-300";

  return (
    <button className={`${baseStyle} ${typeStyle}`} onClick={onClick}>
      {children}
    </button>
  );
}
```

Usage:

```jsx
<Button type="primary">Save</Button>
<Button type="secondary">
  <span role="img" aria-label="cancel">❌</span> Cancel
</Button>
```

---

## 6. Add Default Props or Type Checking (Optional)

Using **PropTypes** or **TypeScript** helps document usage.

**With PropTypes:**

```bash
npm install prop-types
```

```jsx
import PropTypes from "prop-types";

function Button({ children, onClick, type }) {
  return <button className={`btn-${type}`} onClick={onClick}>{children}</button>;
}

Button.propTypes = {
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func,
  type: PropTypes.oneOf(["primary", "secondary"]),
};

Button.defaultProps = {
  type: "primary",
};

export default Button;
```

---

## 7. Reuse Everywhere

Now instead of repeating button styles, just import your `Button`:

```jsx
import Button from "./components/Button";

export default function App() {
  return (
    <div>
      <Button type="primary">Submit</Button>
      <Button type="secondary">Cancel</Button>
    </div>
  );
}
```

---

✅ That’s a reusable React component:

- Encapsulated in its own file
    
- Flexible with props
    
- Accepts children for customization
    
- Centralized styling
    

---

👉 Do you want me to also show you how to make a **reusable layout component** (like a `Card` that wraps content), not just small UI elements?