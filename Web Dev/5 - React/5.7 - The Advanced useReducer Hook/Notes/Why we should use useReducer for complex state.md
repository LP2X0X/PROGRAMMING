---
tags: 
 - react
 - note
 - advance
---

## 1. What’s a “complex state”?

- A state object with **multiple related values** (like `{a, b, c, ...}`)
    
- State that has **different ways to update** depending on user actions
    
- State where updates depend on **previous state**
    

Example:

```js
const [state, setState] = useState({ a: 1, b: 2 });
```

---

## 2. Using `useState` for complex state

You can update with the setter, but you must **manually merge** parts of the object:

```js
setState(prev => ({ ...prev, a: prev.a + 1 }));
setState(prev => ({ ...prev, b: prev.b * 2 }));
```

Problems:

- Easy to forget `...prev`, which **overwrites** the other fields.
    
- As updates grow, the component ends up with a bunch of scattered logic (`setState` calls all over).
    
- Harder to see the “big picture” of what actions can happen to the state.
    

---

## 3. Using `useReducer` for complex state

Instead of calling `setState` everywhere, you define **one reducer function** with clear rules for updating state.

```js
function reducer(state, action) {
  switch (action.type) {
    case "incA":
      return { ...state, a: state.a + 1 };
    case "doubleB":
      return { ...state, b: state.b * 2 };
    default:
      return state;
  }
}

const [state, dispatch] = useReducer(reducer, { a: 1, b: 2 });
```

Usage:

```js
dispatch({ type: "incA" });
dispatch({ type: "doubleB" });
```

Benefits:

- **Centralized update logic** → all changes live in one place.
    
- **Predictable** → every possible action is described in the reducer.
    
- **Scales better** → when state grows, you just add cases, instead of scattering more `setState` calls.
    
- **Easier to test** → you can test the reducer function separately without React.
    

---

## 4. Rule of thumb

- ✅ Use **`useState`** if:
    
    - State is simple (a few values).
        
    - Updates are straightforward.
        
    - Example: a toggle, an input value.
        
- ✅ Use **`useReducer`** if:
    
    - State has **multiple related values**.
        
    - Updates are **complex** (different action types).
        
    - You want a **single place to control how state changes**.
        

---

## 5. Example comparison

### With `useState` (error-prone)

```js
const [form, setForm] = useState({ name: "", age: 0 });

function updateName(name) {
  setForm(prev => ({ ...prev, name })); // need spread
}

function updateAge(age) {
  setForm(prev => ({ ...prev, age })); // need spread
}
```

### With `useReducer` (clearer)

```js
function reducer(state, action) {
  switch (action.type) {
    case "setName":
      return { ...state, name: action.payload };
    case "setAge":
      return { ...state, age: action.payload };
    default:
      return state;
  }
}

const [form, dispatch] = useReducer(reducer, { name: "", age: 0 });
```

---

✅ **Key takeaway**:  
Use `useReducer` when state updates become **too complex or too numerous to manage cleanly with `useState`**.  
It’s not about `{a: x, b: y}` being inherently “complex”, but about how often and in how many ways you’ll update them.