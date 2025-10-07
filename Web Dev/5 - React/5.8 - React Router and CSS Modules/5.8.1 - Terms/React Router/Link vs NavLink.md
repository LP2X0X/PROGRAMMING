---
tags: 
 - react
 - route
 - term
---

## 🔹 `Link`

- Used for **basic navigation**.
    
- Renders an `<a>` tag that updates the URL without reloading the page.
    
- Doesn’t care whether it’s currently active.
    

```jsx
import { Link } from "react-router-dom";

<Link to="/profile">Go to Profile</Link>
```

- Best when you **just want navigation**, nothing fancy.
    

---

## 🔹 `NavLink`

- Used for **navigation items that should reflect “active” state**.
    
- It works just like `Link`, but adds an `active` class (or lets you customize styles) when the link matches the current URL.
    

```jsx
import { NavLink } from "react-router-dom";

<NavLink 
  to="/profile" 
  className={({ isActive }) => isActive ? "active-link" : ""}
>
  Profile
</NavLink>
```

- Useful for menus, sidebars, tabs, etc.
    
- Lets you **style differently when the route is active** (e.g. highlight the current page in the navbar).
    

---

## 🔹 When to use which

- **`Link`** → plain navigation (e.g. buttons, redirects, "read more" links).
    
- **`NavLink`** → navigation UI where the user needs to see which section is active (menus, navbars, tabs).
    

---

👉 Rule of thumb:

- **If you don’t care about active state → `Link`**
    
- **If you want “highlight the current page” → `NavLink`**
    