---
tags: 
 - react
 - context
 - note
---

## ⚛️ When a Context Value Changes

When the value of a React Context changes, **React triggers a re-render** — but **not for the whole tree**.

### ✅ The rules:

1. **Every component that uses `useContext(SomeContext)` will re-render** when the context’s value changes.
    
2. **Components that do not read from that context will _not_ re-render.**
    
3. **The provider component itself** (`<SomeContext.Provider>`) re-renders when its parent re-renders **or** when the `value` prop changes.
    

---

## 🧩 Example

```jsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme } = useContext(ThemeContext);
  return <h1>Header — Theme: {theme}</h1>;
}

function Footer() {
  console.log("Footer render");
  return <h3>Footer</h3>;
}

function App() {
  return (
    <ThemeProvider>
      <Header />
      <Footer />
    </ThemeProvider>
  );
}
```

When you call `setTheme("dark")`:

|Component|Re-render?|Why|
|---|---|---|
|**ThemeProvider**|✅|Its state changed|
|**Header**|✅|Uses `theme` from context|
|**Footer**|❌|Does not consume context|

---

## ⚠️ Common Pitfall

If your provider’s `value` object is **recreated on every render**, all consumers will re-render even if the actual value didn’t change.

Example of bad pattern:

```jsx
<SomeContext.Provider value={{ count, setCount }}>  // ❌ new object every render
```

✅ Fix it with `useMemo`:

```jsx
const value = useMemo(() => ({ count, setCount }), [count]);
<SomeContext.Provider value={value}>
```

Now React will re-render consumers **only when `count` changes**.

---

## 🧠 Summary Table

|Component Type|Re-renders when...|
|---|---|
|Provider|Its parent re-renders or its `value` prop changes|
|Consumer (useContext)|The value from context changes|
|Non-consumer children|Never due to context change|
|Memoized consumers|Only if context value breaks memoization|