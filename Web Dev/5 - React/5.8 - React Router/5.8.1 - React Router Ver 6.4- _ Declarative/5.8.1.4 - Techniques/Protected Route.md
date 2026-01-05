---
tags: 
 - react 
 - technique
 - route
---

## **1. Purpose of a Protected Route**

A **Protected Route** is used to block access to certain pages unless the user is **authenticated**.  

If the user is **not authenticated** and they try to access part of the app using the URL, they will be redirected to a public page (usually `/` or `/login`).

---

# **2. How It Works**

### **Step-by-step logic**

```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();  // Get authentication state
  const navigate = useNavigate();         // Used for redirecting

  useEffect(() => {
    if (!isAuthenticated) {
      navigate("/");                      // Redirect to home if not logged in
    }
  }, [isAuthenticated, navigate]);

  return isAuthenticated ? children : null;
}
```

### **What happens:**

1. **ProtectedRoute renders before the AppLayout.**
    
2. It checks `isAuthenticated` from auth context.
    
3. The `useEffect` runs:
    
    - If **not authenticated** → immediately call `navigate("/")`.
        
4. The component returns:
    
    - `children` (the protected UI) **if authenticated**
        
    - `null` if not (so nothing renders during redirect)
        

---

# **3. Why the Protected Route Wraps AppLayout**

Route setup:

```jsx
<Route
  path="app"
  element={
    <ProtectedRoute>
      <AppLayout />
    </ProtectedRoute>
  }
>
```

Means:

- The whole `/app/*` tree is protected.
    
- All child routes such as:
    
    - `/app/cities`
        
    - `/app/countries`
        
    - `/app/form`
        
    - `/app/cities/:id`
        
    - etc.
        

…can only be accessed when authenticated.

---

# **4. Why Use `useEffect` Instead of Inline Redirect?**

Because React Router v6 removed the old `<Redirect />` component, so the redirection must happen through `navigate()`.

`navigate()` **must not run during render**.  
So you put it inside `useEffect`.

---

# **5. Why Returning `null`?**

When a user is not authenticated:

- You don’t want to render the protected UI
    
- But `navigate("/")` will run inside `useEffect`, not immediately
    
- So you return `null` temporarily until navigation completes
    

---

# **6. Common Behavior Summary**

|Action|User Auth State|Result|
|---|---|---|
|Visit `/app/cities`|authenticated|show `<CityList />`|
|Visit `/app/form`|authenticated|show `<Form />`|
|Visit any `/app/*` route|**not authenticated**|immediately redirected to `/`|

---

# **7. Typical Flow**

**User loads a protected page → ProtectedRoute checks auth →**  
✔️ If authenticated → render page  
❌ If not authenticated → redirect to `/`

---

# **8. Pros of This Pattern**

- Very simple and easy to reason about
    
- Works perfectly with React Router v6
    
- Keeps security logic out of UI components
    
- Reusable across the app
    