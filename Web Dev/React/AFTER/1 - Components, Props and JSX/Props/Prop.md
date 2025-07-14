---
tags: react, term, fundamental
---

### **🔷 What is a Prop in React?**

**Props** (short for **properties**) are how you pass **data** from one component to another in React. They’re read-only and allow components to be **reusable and dynamic**.

```ad-note
Anything can be passed as props.
```

---

### **✅ Basic Example**

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

Usage:

```jsx
<Welcome name="Long" />
```

- name="Long" is a **prop**
- props.name inside the component gets its value

---

### **✅ Destructuring Props**

A cleaner and more common way:

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

---

### **✅ Multiple Props**

```jsx
function Info({ name, age }) {
  return <p>{name} is {age} years old.</p>;
}

<Info name="Long" age={24} />
```

---

### **✅ Props Can Be Any Type**

|**Type**|**Example**|
|---|---|
|String|\<Card title="Hello" />|
|Number|\<Counter value={5} />|
|Boolean|\<Modal isOpen={true} />|
|Array|\<List items={[1, 2, 3]} />|
|Object|\<User data={{ id: 1, name: "A" }} />|
|Function|\<Button onClick={handleClick} />|

---

### **✅ Default Props**

You can set fallback values:

```jsx
function Greeting({ name = "Guest" }) {
  return <h1>Hello, {name}</h1>;
}
```

---

### **✅ Props Are Read-Only**

You **cannot** (and should not) modify props inside a component:

```jsx
function Example({ message }) {
  // ❌ message = "New message"; // don't do this
  return <p>{message}</p>;
}
```

If you want to change values, use **state** instead.

---

### **✅ Passing Children with** 

### **props.children**

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

// Usage:
<Card>
  <h1>Title</h1>
  <p>Content here</p>
</Card>
```