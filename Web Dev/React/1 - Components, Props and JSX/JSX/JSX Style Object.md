---
tags:
  - react
  - jsx
  - fundamental
---

In **JSX**, CSS styles are written as **JavaScript objects**, not strings. This is because JSX is JavaScript — not HTML.

---

### **✅ Basic Syntax**

```jsx
const style = {
  color: "red",
  backgroundColor: "black",
};

function MyComponent() {
  return <h1 style={style}>Hello</h1>;
}
```

---

### **✅ Inline Style (without a variable)**

```jsx
function MyComponent() {
  return (
    <h1 style={{ color: "blue", fontSize: "24px" }}>
      Hello
    </h1>
  );
}
```

---

### **🔸 Key Differences from Regular CSS**

|**Regular CSS**|**JSX Style Object**|
|---|---|
|background-color|backgroundColor|
|font-size|fontSize|
|text-align|textAlign|

- JSX uses **camelCase**
- Values like "24px" or "1.5em" are passed as **strings**

---

### **✅ Dynamic Styles**

```jsx
function Alert({ isError }) {
  return (
    <p
      style={{
        color: isError ? "red" : "green",
        fontWeight: "bold",
      }}
    >
      {isError ? "Error!" : "Success!"}
    </p>
  );
}
```

---

### **⚠️ Limitations**

- JSX style is good for **dynamic, component-specific styles**, but not ideal for large styling.
    
- For global styles or reusable styles, use:
    - CSS files
    - CSS Modules
    - Tailwind CSS
    - Styled-components (CSS-in-JS)