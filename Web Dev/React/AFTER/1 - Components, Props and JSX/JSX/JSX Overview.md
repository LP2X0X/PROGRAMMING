---
tags: react, summary, fundamental
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
|Self-closing Tags|<img />, <input />|
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

### **✅ JSX Rules**

1. **Return one root element**

```jsx
return (
  <div>
    <h1>Hello</h1>
    <p>World</p>
  </div>
);
```

2. **Use className instead of class**

```jsx
<div className="box">Hello</div>
```

3. **Use camelCase for attributes**
    
    Like onClick, htmlFor, tabIndex, etc.
    