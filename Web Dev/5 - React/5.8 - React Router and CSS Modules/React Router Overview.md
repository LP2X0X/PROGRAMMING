---
tags: 
 - react
 - router
 - library
 - overview
---

## 🚦 React Router Overview

### 🔑 What It Is

React Router is a **library for routing in React applications**.  
It enables **client-side routing**, meaning the UI updates when the URL changes **without refreshing the page**.

---

### ⚡ Core Concepts

1. **Router**
    
    - The wrapper that enables routing in the app.
        
    - Commonly used:
        
        - `<BrowserRouter>` → uses the browser’s history API (most common).
            
        - `<HashRouter>` → uses URL hashes (`/#/about`) for older setups.
            
2. **Routes & Route**
    
    - Define URL → Component mappings.
        
    - Example:
        
        ```jsx
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
        ```
        
3. **Link / NavLink**
    
    - Used for navigation without reloading.
        
    - `NavLink` can apply active styling.
        
    - Example:
        
        ```jsx
        <Link to="/about">About</Link>
        <NavLink to="/about" className="active">About</NavLink>
        ```
        
4. **useNavigate (previously useHistory)**
    
    - Navigate programmatically in code.
        
    - Example:
        
        ```jsx
        const navigate = useNavigate();
        navigate("/login");
        ```
        
5. **useParams**
    
    - Access route parameters (dynamic routes).
        
    - Example: `/user/:id` → `useParams().id`.
        
6. **Nested Routes**
    
    - Define routes inside other routes, useful for layouts or dashboards.
        
    - Example:
        
        ```jsx
        <Route path="dashboard" element={<Dashboard />}>
          <Route path="stats" element={<Stats />} />
        </Route>
        ```
        
7. **Outlet**
    
    - A placeholder component for rendering child routes in nested routing.
        
8. **Protected / Private Routes**
    
    - Custom logic to allow or block access (e.g., only logged-in users can see certain pages).
        

---

### 🖼️ Example

```jsx
import { BrowserRouter, Routes, Route, Link, useParams, useNavigate } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link> | <Link to="/about">About</Link> | <Link to="/user/42">User 42</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/user/:id" element={<User />} />
      </Routes>
    </BrowserRouter>
  );
}

function Home() {
  return <h2>Home Page</h2>;
}

function About() {
  return <h2>About Page</h2>;
}

function User() {
  const { id } = useParams();
  return <h2>User ID: {id}</h2>;
}
```

---

### 🚀 Why Use React Router?

- SPA navigation without reload.
    
- Handles nested routes and layouts.
    
- Works with browser history (back/forward buttons).
    
- Easy to protect routes (authentication).
    
- Syncs UI with URL for deep linking & bookmarks.
    