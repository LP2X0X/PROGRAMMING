---
tags:
  - react
  - note
  - fundamental
---

- State management includes:
	- Deciding **when** to create pieces of state.
	- What **type** of state are necessary.
	- **Where** to place each piece of state.
	- How data **flows** through app.

----

## Types of state
### **🌐 Global State**

#### **📌 What is it?**

- State that is **shared across multiple components**.
- Lives **outside** any single component.
- Usually managed using:
    - React Context API
    - Third-party libraries (Zustand, Redux, Recoil, Jotai, etc.)

### **✅ Example (Context API):**

```jsx
const ThemeContext = createContext();

function App() {
  const [darkMode, setDarkMode] = useState(false);

  return (
    <ThemeContext.Provider value={{ darkMode, setDarkMode }}>
      <Header />
      <Main />
    </ThemeContext.Provider>
  );
}

function Header() {
  const { darkMode, setDarkMode } = useContext(ThemeContext);
  return (
    <button onClick={() => setDarkMode((d) => !d)}>
      Toggle Theme ({darkMode ? "Dark" : "Light"})
    </button>
  );
}
```

### **🧠 When to use:**

- User authentication data
- Theme toggling (dark/light)
- Language/locale settings
- Cart items in an e-commerce app
- Any data that multiple components depend on

---

### **🏝️ Local State**

#### **📌 What is it?**

- State that **belongs to a single component**.
- Managed using useState, useReducer, etc.
- It only affects the component it’s defined in (and optionally its children).

#### **✅ Example:**

```jsx
function Counter() {
  const [count, setCount] = useState(0); // local to Counter

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

#### **🧠 When to use:**

- UI toggles (isModalOpen)
- Form input values
- Component-specific logic

--- 
#### **🔁 Summary Table**

|**Feature**|**Local State**|**Global State**|
|---|---|---|
|Scope|One component|Multiple components|
|Stored in|useState, useReducer|Context API, Redux, Zustand, etc.|
|Easy to manage|✅|❌ (can get complex)|
|Suitable for|Forms, modals, toggles|Auth, themes, user settings|

----

## When and Where?
![[Pasted image 20250723185241.png|center]]

----

##  Lifting state up
![[Pasted image 20250723191119.png|center]]
![[Pasted image 20250723191625.png|center]]