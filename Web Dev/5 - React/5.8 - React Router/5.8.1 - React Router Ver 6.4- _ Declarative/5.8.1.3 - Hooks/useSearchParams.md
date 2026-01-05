---
tags: 
 - react
 - router
 - hook
---

`useSearchParams` is a React Router hook used to **read and write URL query parameters** (the part after `?` in a URL).  
It behaves similarly to the Web API `URLSearchParams`, but is integrated with React Router’s navigation model.

Example URL:

```
/products?page=2&sort=price
```

---

# What It Returns

`useSearchParams()` returns a tuple:

```js
const [searchParams, setSearchParams] = useSearchParams();
```

1. **`searchParams`**  
    A `URLSearchParams` object.  
    You use it to read values:
    
    ```js
    searchParams.get("page"); // "2"
    searchParams.get("sort"); // "price"
    ```
    
2. **`setSearchParams`**  
    A function to update the query string, triggering navigation and component re-render:
    
    ```js
    setSearchParams({ page: 3, sort: "name" });
    // URL becomes: ?page=3&sort=name
    ```
    

---

# Typical Use Cases

1. **Pagination**
    
    ```
    /users?page=3
    ```
    
2. **Sorting and filtering**
    
    ```
    /products?sort=price&min=10&max=50
    ```
    
3. **Search keywords**
    
    ```
    /search?query=javascript
    ```
    
4. **Managing UI state via URL**  
    Tabs, view modes, toggles.
    

Query params are ideal because they preserve state across refresh, shareable URLs, and browser navigation.

---

# How to Read Search Params

```jsx
const [searchParams] = useSearchParams();

const page = searchParams.get("page") || "1";
const sort = searchParams.get("sort") || "default";
```

If a parameter is missing, `.get()` returns `null`.

---

# How to Update (Write) Search Params

You can replace all parameters:

```jsx
setSearchParams({ page: 4, sort: "name" });
```

Or modify existing ones:

```jsx
const [searchParams, setSearchParams] = useSearchParams();

searchParams.set("page", "2");
searchParams.set("sort", "price");

setSearchParams(searchParams);
```

Or append:

```js
searchParams.append("filter", "active");
setSearchParams(searchParams);
```

---

# Important Details

### 1. Values are always **strings**

If you need numbers, convert manually:

```js
const page = Number(searchParams.get("page"));
```

### 2. Changing search params = navigation

It updates the URL and triggers rerender (like `navigate()`).

### 3. Order of keys does not matter

React Router re-serializes them deterministically.

### 4. State persists through refresh because it lives in the URL

No need for component-level state or global state.

---

# Best Practices

1. **Use search params for UI options**, not core identity  
    Example: page, sort, filter, view mode.
    
2. **Use path params (`:id`) for entity identity**  
    Example: `/users/:userId` not `?userId=5`.
    
3. **Keep default values in code**  
    URLs should not break your component.
    
4. **Avoid mixing search params and local state unnecessarily**  
    Keep the source of truth consistent.
    

---

# Summary

`useSearchParams` is the React Router hook for query string management.  
It gives you:

- an easy reader (`searchParams`)
    
- a navigator-friendly writer (`setSearchParams`)
    
- consistent and shareable UI state management using URLs
    