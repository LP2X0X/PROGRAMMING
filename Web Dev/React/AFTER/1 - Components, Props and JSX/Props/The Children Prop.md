---
tags: react, term, fundamental
---

In React, **the children prop** is a special built-in prop that represents **whatever you put between a component’s opening and closing tags**.

---

### **🔹 Basic Example**

```jsx
function Wrapper({ children }) {
  return <div className="wrapper">{children}</div>;
}

export default function App() {
  return (
    <Wrapper>
      <p>Hello World</p>
      <button>Click me</button>
    </Wrapper>
  );
}
```

### **✅ What Happens**

- Inside the Wrapper component, the children prop contains:
	```jsx
	<>
	  <p>Hello World</p>
	  <button>Click me</button>
	</>
	```

- So Wrapper will render:
	```jsx
	<div class="wrapper">
	  <p>Hello World</p>
	  <button>Click me</button>
	</div>
	```

---

### **🔹 Why It’s Useful**

- It lets components **wrap other content**.
- You can build reusable layouts like cards, modals, or layout containers.

---

### **🔹 How to Use children**

- Just include {children} in your component’s return JSX:
	```jsx
	function Box({ children }) {
	  return <section className="box">{children}</section>;
	}
	```

- Then use:
	```jsx
	<Box>
	  <h1>Title</h1>
	  <p>Some content inside the box</p>
	</Box>
	```

---

### **🔹 Technical Note**
- children can be anything: text, JSX, numbers, an array of nodes, etc.
- You don’t need to pass it manually — React does that automatically.