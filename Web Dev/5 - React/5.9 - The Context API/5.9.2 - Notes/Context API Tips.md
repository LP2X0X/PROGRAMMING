---
tags: 
 - react
 - context
 - api
 - tips
---

React’s **Context API** allows you to share state or functions across your component tree without passing props manually.  
By combining it with **custom hooks**, you make your context usage cleaner, safer, and easier to maintain.

---

### ⚙️ **Core Pattern**

1. **Create the Context**
    
    ```jsx
    const ExampleContext = createContext();
    ```
    
2. **Wrap Components in a Provider**
    
    ```jsx
    function ExampleProvider({ children }) {
      const [state, setState] = useState();
      const value = { state, setState };
    
      return (
        <ExampleContext.Provider value={value}>
          {children}
        </ExampleContext.Provider>
      );
    }
    ```
    
3. **Access with a Custom Hook**
    
    ```jsx
    function useExample() {
      const context = useContext(ExampleContext);
      if (!context) // Check context
        throw new Error("useExample must be used within an ExampleProvider");
      return context;
    }
    ```
    
4. **Use Anywhere**
    
    ```jsx
    function ExampleComponent() {
      const { state, setState } = useExample();
      return <div>{state}</div>;
    }
    ```
    

---

### 🧠 **Why This Pattern Is Cleaner**

|Benefit|Description|
|---|---|
|**Encapsulation**|All logic for reading and validating context is inside the custom hook.|
|**Cleaner imports**|Components only import one hook instead of both the context and `useContext`.|
|**Error safety**|Prevents components from using context outside its Provider.|
|**Separation of concerns**|The Provider focuses on state management; the hook focuses on access.|
|**Easy scaling**|You can have multiple contexts (Auth, Theme, Posts, etc.) with consistent structure.|

---

### 🧱 **Why Wrap with Providers**

Wrapping parts of your app with context Providers keeps data **localized** and avoids **prop drilling**:

```jsx
<AuthProvider>
  <ThemeProvider>
    <SettingsProvider>
      <App />
    </SettingsProvider>
  </ThemeProvider>
</AuthProvider>
```

Each Provider manages its own domain (auth, theme, settings), making the app easier to reason about and refactor.

---

### ✅ **Best Practices**

- Use **one custom hook per context** (`useAuth`, `useTheme`, `usePosts`, etc.).
    
- Always **guard** against missing Providers inside the hook.
    
- Keep Provider logic **focused** — avoid mixing unrelated states.
    
- Group Providers at the root or near where they’re needed.
    
- Prefer **derived state** (computed from existing data) over duplicating values.
    

---

### 🧭 **Summary**

|Concept|Purpose|
|---|---|
|**Context Provider**|Supplies shared state or actions to descendant components|
|**`useContext()`**|Retrieves the nearest context value|
|**Custom Hook**|Wraps `useContext` for clean, consistent, and safe usage|
|**Result**|Cleaner code, better modularity, easier testing, and no prop drilling|