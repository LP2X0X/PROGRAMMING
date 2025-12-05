---
tags: 
 - react
 - router
 - hook
---

`useLocation` is a React Router hook that gives you the **current location object** of the app.  
The “location” describes **where the user is now**, including:

- `pathname` — the URL path
    
- `search` — the query string (`?key=value`)
    
- `hash` — the fragment (`#section`)
    
- `state` — arbitrary state passed during navigation
    
- `key` — a unique identifier for each navigation
    

It updates every time the URL changes.

---

# Basic Usage

```jsx
import { useLocation } from "react-router-dom";

function MyComponent() {
  const location = useLocation();

  console.log(location);

  return <div>Current path: {location.pathname}</div>;
}
```

---

# Location Object Structure

`useLocation()` returns an object like:

```js
{
  pathname: "/products/123",
  search: "?sort=price",
  hash: "#details",
  state: { from: "homepage" },
  key: "a1b2c3"
}
```

### Explanation:

### 1. `pathname`

The path portion of the URL:

```
/users/5/settings
```

### 2. `search`

Everything after `?`, including the `?`.

```
?sort=price&min=100
```

### 3. `hash`

Everything after `#`:

```
#top
#section2
```

### 4. `state`

Optional JS object only available through navigation:

```jsx
navigate("/login", { state: { from: "/checkout" } });
```

You can read it with:

```js
location.state.from
```

### 5. `key`

Internal unique ID used by React Router for navigation history.

---

# When to Use `useLocation`

### 1. **Detect route changes**

Useful for analytics, animations, transitions.

```jsx
useEffect(() => {
  console.log("Route changed!");
}, [location]);
```

### 2. **Conditional rendering based on route**

Example: hide a header on certain pages.

```jsx
if (location.pathname === "/login") return null;
```

### 3. **Read query string manually**

Although `useSearchParams` is usually better:

```js
new URLSearchParams(location.search).get("page");
```

### 4. **Access `state` passed during navigation**

Example of sending state:

```js
navigate("/checkout", { state: { items: cart } });
```

Reading:

```js
const location = useLocation();
const items = location.state?.items;
```

### 5. **Highlight active sidebar or menu items**

```js
location.pathname.startsWith("/dashboard")
```

---

# Important Notes

1. **`location` always changes when the URL changes**, even if the path is the same but the search string changes.
    
2. **`state` is not persistent across refresh**  
    It is stored in memory, not the URL.
    
3. **Never mutate the location object directly**  
    It is read-only.
    
4. **Re-renders occur on every navigation**, so use it carefully inside heavy components.
    

---

# Summary

`useLocation` gives you the **current URL information** in a structured object.  
Use it when you need to:

- inspect path / query / hash
    
- detect navigation changes
    
- read state passed through `navigate`
    
- conditionally render UI based on URL
    

For manipulating search parameters, prefer `useSearchParams`.  
For reading path parameters, use `useParams`.