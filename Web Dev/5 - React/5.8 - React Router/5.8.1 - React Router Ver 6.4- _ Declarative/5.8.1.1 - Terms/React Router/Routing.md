---
tags: 
 - react
 - term
 - advance
---

In **React**, the term **routing** usually refers to handling navigation between different "pages" or "views" in a [[Single Page Application]] (SPA). Since React itself does not include routing by default, developers commonly use a library like **React Router** to implement it.

- 👉 With routing, we match **different URLs to different UI views** (React components): **routes**
    
- 👉 This enables users to **navigate between different applications screens**, using the browser URL
    
- 👉 Keeps the **UI in sync** with the current browser URL
    

````ad-note
**`react-router-dom` is _not included by default_** when you create a new React app — **neither with Create React App nor with Vite**.

To install it, use:
```bash
npm install react-router-dom
```
````

---

### 📝 Detailed Notes

1. **Client-side routing**
    
    - Traditional websites use **server-side routing**: each time you click a link, the browser sends a request to the server, which returns a new HTML page.
        
    - In React, routing is **handled on the client**. The app dynamically swaps components based on the URL **without reloading the page**. This makes navigation much faster.
        
2. **Matching URLs to UI views (routes)**
    
    - A **route** connects a **URL path** (like `/about`) to a **React component** (like `<About />`).
        
    - Example:
        
        ```jsx
        <Route path="/about" element={<About />} />
        ```
        
3. **Navigation between screens**
    
    - Users can click on links or buttons to move around the app.
        
    - Instead of refreshing, React changes which component is shown.
        
    - Example:
        
        ```jsx
        <Link to="/contact">Go to Contact Page</Link>
        ```
        
4. **Syncing UI with browser URL**
    
    - The displayed component always matches the current URL in the browser.
        
    - This ensures:
        
        - **Back/forward buttons** work correctly.
            
        - **Direct URL access** works (e.g., entering `/profile` in the browser opens the Profile page).
            
        - **Bookmarking** works (users can save and revisit the exact page).
            

---

### 🔑 Basic Terms in React Routing

1. **Route**
    
    - Defines a mapping between a URL path and a component.
        
    - Example: `/about` → `<AboutPage />`.
        
2. **Router**
    
    - The wrapper that enables routing functionality.
        
    - Example: `<BrowserRouter>` keeps track of the URL and re-renders components accordingly.
        
3. **Link / NavLink**
    
    - Used instead of `<a>` tags to navigate between pages **without refreshing** the browser.
        
    - Example: `<Link to="/about">About</Link>`.
        
4. **Switch / Routes**
    
    - Ensures only one route matches at a time.
        
    - In React Router v6, `Routes` replaced `Switch`.
        
5. **useNavigate / useHistory**
    
    - A hook for programmatic navigation (navigating via code instead of clicking a link).
        
    - Example: `navigate("/login")`.
        
6. **Dynamic Route**
    
    - Route with parameters (e.g., `/user/:id`) that you can read with `useParams()`.
        
7. **Nested Routes**
    
    - Routes inside other routes, useful for layouts or dashboards.
        
8. **Route Guards (Private Routes)**
    
    - Custom logic that restricts access to certain routes (like requiring login).
        

---

### ⚡ Example (React Router v6)

```jsx
import { BrowserRouter as Router, Routes, Route, Link } from "react-router-dom";

function App() {
  return (
    <Router>
      <nav>
        <Link to="/">Home</Link> | <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/user/:id" element={<User />} />
      </Routes>
    </Router>
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