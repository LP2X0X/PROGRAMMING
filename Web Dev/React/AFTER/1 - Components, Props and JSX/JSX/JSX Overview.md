---
tags: react, jsx, summary, fundamental
---

### **🔷 What is JSX in React?**

**JSX (JavaScript XML)** is a syntax extension for JavaScript used in **React**. It allows you to embed JavaScript, CSS and HTML and React components into HTML, making UI code more readable and expressive.

- Browsers don't understand JSX out of the box, so you'll need a JavaScript compiler, such as a Babel, to transform your JSX code into regular JavaScript.

![[Pasted image 20250703201910.png|center|400]]

---

### **✅ Basic Example**

```jsx
const element = <h1>Hello, world!</h1>;
```

This looks like HTML, but it’s actually JavaScript! React transforms it under the hood into:

```jsx
const element = React.createElement('h1', null, 'Hello, world!');
```

---

### **✅ JSX in a Component**

- JSX works essentially like HTML, but we can enter "JavaScript mode" by using { }. We can place JavaScript **expressions** in side the { }.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

Used like this:

```jsx
<Welcome name="Long" />
```

---

### **✅ Key Features of JSX**

|**Feature**|**Example**|
|---|---|
|Embed JavaScript|{} inside JSX|
|Self-closing Tags|\<img />, \<input />|
|Props|\<Component name="value" />|
|Children|\<Component>Some content\</Component>|
|Conditional Render|{isLoggedIn ? <User /> : <Guest />}|
|Lists|{items.map(item => \<li>{item}\</li>)}|

---

### **✅ Embedding Expressions**

```jsx
const name = "Long";
const element = <h1>Hello, {name}</h1>;
```

---

### **✅ Styling in JSX**

Inline styles use an object:

```jsx
const style = {
  color: 'blue',
  fontSize: '20px',
};

const element = <h1 style={style}>Hello!</h1>;
```

---

## **✅ JSX Rules**

### ✅ **1. Return One Root Element**

### 🔸 What it means:

Every JSX component must return a **single parent/root element**.

### 🔸 Why:

Because JSX compiles to `React.createElement(...)`, which expects **one root node**. Returning multiple root elements causes a syntax error.

#### ✅ Correct:

```jsx
function App() {
  return (
    <div>
      <h1>Hello</h1>
      <p>World</p>
    </div>
  );
}
```

#### ❌ Incorrect:

```jsx
function App() {
  return (
    <h1>Hello</h1>
    <p>World</p>   // ❌ Cannot return two siblings at top-level
  );
}
```

#### 🛠 Solution:

* Wrap everything in a `<div>` or `<section>`
* Or use **[[React Fragment|React Fragment]]**: `<></>`

```jsx
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

---

### ✅ **2. Use `className` Instead of `class`**

#### 🔸 What it means:

In JSX, you must use `className` (not `class`) to apply CSS classes.

#### 🔸 Why:

* `class` is a **reserved keyword in JavaScript** (used for class declarations).
* React uses `className` to avoid confusion and errors during compilation.

#### ✅ Correct:

```jsx
<div className="header">Welcome</div>
```

#### ❌ Incorrect:

```jsx
<div class="header">Welcome</div>  // ❌ JSX parser will complain
```

> Behind the scenes, React sets `class` on the DOM element, but you must write `className` in JSX.

---

### ✅ **3. Use camelCase for Attributes**

#### 🔸 What it means:

Use **camelCase** instead of lowercase for most JSX HTML attributes.

| HTML       | JSX        |
| ---------- | ---------- |
| `onclick`  | `onClick`  |
| `tabindex` | `tabIndex` |
| `for`      | `htmlFor`  |
| `readonly` | `readOnly` |

#### 🔸 Why:

JSX is **JavaScript**, and in JavaScript we use **camelCase** for properties and event handlers.

#### ✅ Examples:

```jsx
<button onClick={handleClick}>Click me</button>
<label htmlFor="email">Email</label>
<input type="text" readOnly />
```

> These camelCase props are transformed by React into the correct DOM attributes.

---

### ✅ Summary

| Rule                           | Why It Matters                                                 |
| ------------------------------ | -------------------------------------------------------------- |
| One root element               | JSX compiles to a single function call (`React.createElement`) |
| `className` instead of `class` | Avoids conflict with JS `class` keyword                        |
| camelCase attributes           | Follows JS conventions; avoids syntax errors                   |