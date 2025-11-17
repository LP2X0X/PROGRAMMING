---
tags: 
 - react
 - note
 - input
 - ref
---

# ✅ 1. Why using `ref` avoids re-render

If you do this:

```jsx
const [value, setValue] = useState("");
```

and bind it:

```jsx
<input value={value} onChange={e => setValue(e.target.value)} />
```

Every time the user types:

- `setValue` runs
    
- React re-renders the component
    
- This is called a **controlled input**
    

But if you use a ref:

```jsx
const inputRef = useRef();

<input ref={inputRef} />
```

Typing _does not change any React state_, so:

- **no re-renders**
    
- input updates itself (because DOM handles input value)
    

This is called an **uncontrolled input**.

---

# ✅ 2. Example: reading input value WITHOUT re-render

```jsx
function Form() {
  const inputRef = useRef();

  function handleSubmit() {
    console.log(inputRef.current.value); 
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

Here:

- Typing does NOT trigger updates
    
- Input changes normally (browser behavior)
    
- React component stays frozen unless props change
    

---

# ✅ 3. Example: writing to the input without re-render

```jsx
function Form() {
  const inputRef = useRef();

  function clearInput() {
    inputRef.current.value = "";
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={clearInput}>Clear</button>
    </>
  );
}
```

`inputRef.current.value = ""` updates the DOM  
but **React does not re-render**.

---

# 🧠 When should you use refs for inputs?

Use `ref` when:

### ✔ You don’t need re-render on each keystroke

Examples:

- reading input on form submit
    
- controlling a search bar with debounce
    
- implementing “focus input after submit”
    
- clearing input
    
- validating input when button clicked
    
- measuring input width
    

### ❌ Do not use refs when:

- you need React to react to each keystroke
    
- you want the value part of UI state
    
- you need derived values (e.g., uppercase, validation messages)
    

---

# 📌 Why using `ref` avoids re-renders in JS terms

Because:

- React re-renders only when **state** or **props** change
    
- `ref.current` is just a normal **mutable object**
    
- Updating `.current` does NOT trigger any state change
    
- So React does nothing — no re-render
    

Pure JS explanation:

```js
inputRef.current.value = "hey";
// This modifies an object property → NOT React state
// So React ignores it and does not schedule render
```