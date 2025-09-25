---
tags:
  - react
  - term
  - fundamental
---

- React application are entirely made out of **components**. It is the building block of user interface.
	![[Pasted image 20250703191853.png|center|500]]
- Component has its own data (which made out of [[Prop|props]] and state), logic and appearance. It can then be reused, nested inside each other, and pass data between them.
	![[Pasted image 20250704205115.png|center|500]]
- React component names must always start with a capital letter, while HTML tags must be lowercase.

---

### **✅ 1. Function Component (Modern Way)**

```js
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

Or using arrow function:

```js
const Welcome = ({ name }) => <h1>Hello, {name}!</h1>;
```

---

### **✅ 2. Using a Component**

```js
function App() {
  return <Welcome name="Long" />;
}
```

---

### **✅ 3. Class Component (Older Way)**

```js
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

---

### **✅ 4. Component Types**

|**Type**|**Description**|
|---|---|
|Functional|Most common; just functions that return JSX.|
|Class|Old style with lifecycle methods (componentDidMount, etc.).|
|Presentational|Just display UI; don’t handle logic/state.|
|Container/Smart|Handle data fetching, state, and logic.|
|Higher-Order (HOC)|A function that takes a component and returns a new component.|
|Hooks-based|Functional component using React hooks (useState, useEffect, etc.).|

---

### **✅ 5. JSX Inside Components**

React components return [[JSX Overview|JSX]] (a syntax extension of JavaScript that looks like HTML):

```js
function Header() {
  return (
    <header>
      <h1>My App</h1>
    </header>
  );
}
```

```ad-note
The return value **must be a valid JavaScript <u>expression</u>** (usually JSX itself).
```

---

### **✅ 6. Props and State**

- **Props**: Passed in like attributes.
- **State**: Managed inside the component (only in class components or with hooks).
  
Example with useState:

```js
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```