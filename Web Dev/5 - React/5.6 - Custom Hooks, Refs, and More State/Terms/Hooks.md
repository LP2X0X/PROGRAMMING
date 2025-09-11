---
tags:
  - react
  - term
  - hook
  - fundamental
---

# **⚛️ React Hooks Overview**

React **hooks** are special functions that let you use **React features** (like state and lifecycle) in **functional components** (or "hook" into React internals).

> Hooks were introduced in React 16.8 to avoid needing class components for advanced features.

---

## **📌 Basic Hooks**

### **useState()**

Manages local component **state**.

```js
const [count, setCount] = useState(0);
```

- count: current state value
- setCount(): function to update it

---

### **useEffect()**

Runs **side effects** after render (e.g. fetching data, timers).

```js
useEffect(() => {
  // runs after render
  return () => {
    // cleanup (runs on unmount or before re-run)
  };
}, [dependencies]); // runs only when dependencies change
```

---

### **useRef()**

Stores a **mutable reference** that survives re-renders. Often used to reference DOM elements.

```js
const inputRef = useRef();

<input ref={inputRef} />
```

---

### **useContext()**

Reads a value from a **React Context** (like theme, user, settings).

```js
const theme = useContext(ThemeContext);
```

---

## **🧠 Performance Hooks**

### **useMemo()**

Memoizes the **result of a computation**.

```js
const result = useMemo(() => heavyCalc(a, b), [a, b]);
```

### **useCallback()**

Memoizes a **function** so it doesn’t get re-created unless dependencies change.

```js
const handleClick = useCallback(() => {
  doSomething();
}, []);
```

---

## **☝️ Rules of hooks**
![[Pasted image 20250909185258.png|center|500]]

---

## **🧩 Custom Hooks**

You can build your own hooks to reuse logic across components.

```js
function useCounter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}
```

---

## **📚 Summary Table**

|**Hook**|**Purpose**|
|---|---|
|useState|Add local state to a function component|
|useEffect|Perform side effects (fetch, timer, etc.)|
|useRef|Keep mutable values or access DOM|
|useContext|Access context values|
|useMemo|Cache expensive computed values|
|useCallback|Cache functions to avoid re-renders|