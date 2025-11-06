---
tags: 
 - react
 - hook
 - fundamental
---

## 1. What is `useReducer`?

`useReducer` is a React Hook that’s an **alternative to `useState`** for managing state.  
It’s especially useful when:

- State is **complex** (nested objects, multiple related values).
    
- You have **many ways** to update the state (lots of actions).
    
- You want to keep **state logic separate** from the UI.
    

It follows the **reducer pattern** (same idea as Redux).


---

## 2. Basic syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- `state`: current state value.
    
- `dispatch`: a function you call with an _action_ object (e.g., `{ type: "increment" }`).
    
- `reducer`: a pure function `(state, action) => newState`.
    
- `initialState`: the starting state.
    

![[Pasted image 20250914145858.png|center]]

````ad-tip
It's always a good idea to model the action type as event. Here's an example:
```jsx
function reducer(state, action) {
  switch (action.type) {
    case "city/loaded": // In past tense
      return {
        ...state,
        isLoading: false,
        currentCity: action.payload,
      };

    case "cities/loaded":
      return {
        ...state,
        isLoading: false,
        cities: action.payload,
      };

    case "cities/created":
      return {
        ...state,
        isLoading: false,
        cities: [...state.cities, action.payload],
      };

    case "cities/deleted":
      return {
        ...state,
        isLoading: false,
        cities: state.cities.filter((city) => city.id !== action.payload),
      };

    case "loading":
      return { ...state, isLoading: true };

    case "doneLoading":
      return { ...state, isLoading: false };

    case "rejected":
      return { ...state, isLoad

```
````


---

## 3. How reducer update state

![[Pasted image 20250914150243.png|center]]

---

## 4. Example: simple counter

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error("Unknown action");
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </div>
  );
}
```

---

## 5. When to use `useReducer` vs `useState`

- ✅ `useState`: simple state, independent values.
    
    ```jsx
    const [name, setName] = useState("");
    const [age, setAge] = useState(0);
    ```
    
- ✅ `useReducer`: complex state, multiple actions, or you want all updates centralized.
    
    ```jsx
    const [state, dispatch] = useReducer(reducer, { name: "", age: 0 });
    ```
    

![[Pasted image 20250914145553.png|center|700]]

---

## 6. Lazy initialization (bonus!)

You can provide a function to initialize state **only once**:

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function reducer(state, action) {
  switch (action.type) {
    case "reset":
      return init(action.payload);
    case "increment":
      return { count: state.count + 1 };
    default:
      throw new Error();
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "reset", payload: initialCount })}>
        Reset
      </button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
```

---

✅ **Key takeaway**:  
`useReducer` is just like `useState` but with a **clear structure** for managing more complex or related state updates.