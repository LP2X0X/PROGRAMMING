---
tags:
  - react
  - fundamental
---

In **JSX**, you can apply **inline styles** using a **JavaScript object**. This is often used for quick styling or dynamic styles in React components.

---

### **✅ Basic Syntax**

```jsx
const styleObj = {
  color: 'blue',
  backgroundColor: 'lightgray',
  fontSize: '20px',
};

function StyledComponent() {
  return <h1 style={styleObj}>Hello, styled world!</h1>;
}
```

- The style attribute expects a **JavaScript object**, not a CSS string.
- Property names use **camelCase** instead of hyphens.
    - ✅ backgroundColor
    - ❌ background-color

---

### **✅ Inline Object Directly**

```jsx
function App() {
  return (
    <div style={{ color: 'red', padding: '10px' }}>
      Inline style example
    </div>
  );
}
```

---

### **✅ Dynamic Styles**

You can also compute styles conditionally:

```jsx
function Button({ primary }) {
  const style = {
    backgroundColor: primary ? 'blue' : 'gray',
    color: 'white',
    padding: '10px 20px',
    border: 'none',
  };

  return <button style={style}>Click Me</button>;
}
```

---

### **✅ Using CSS Units**

Even though it’s JavaScript, CSS **unit values must be strings** if they’re not unitless:

```jsx
const boxStyle = {
  width: '100px',    // ✅ use string
  height: 50,        // ✅ some properties like lineHeight or zIndex accept numbers
  borderRadius: '10px',
};
```

---

### **✅ Example: Toggle Theme**

```jsx
function ThemeBox({ isDark }) {
  const style = {
    backgroundColor: isDark ? '#333' : '#fff',
    color: isDark ? '#fff' : '#000',
    padding: '20px',
    borderRadius: '8px',
  };

  return <div style={style}>Theme Box</div>;
}
```