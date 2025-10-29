---
tags: 
 - react
 - router 
 - component
 - route
---

### 🧩 **`<Route>` Overview**

**Purpose:**  
Defines a **mapping** between a **URL path** and a **React element (component)** to render.

---

### 🧱 **Basic Syntax**

```jsx
<Route path="/about" element={<About />} />
```

- `path`: the URL pattern to match.
    
- `element`: what to render when the path matches. This usually is a JSX element because we usually pass props into the component itself.
    

---

### ⚙️ **Where to Use**

Must be inside a `<Routes>` block:

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

---

### 🔄 **Dynamic Parameters**

```jsx
<Route path="/users/:id" element={<User />} />
```

Use with `useParams()`:

```jsx
const { id } = useParams();
```

---

### 🧭 **Nested Routes**

```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route path="settings" element={<Settings />} />
</Route>
```

Render nested routes using `<Outlet />` in `Dashboard`.

---

### 🚫 **Wildcard / Fallback**

```jsx
<Route path="*" element={<NotFound />} />
```

---

### 🧠 **Summary Table**

|Prop|Description|Example|
|---|---|---|
|`path`|URL to match|`/about`, `/users/:id`|
|`element`|Component to render|`<Home />`|
|`index`|Marks default child route|`<Route index element={<Home />} />`|
|`children`|Nested `<Route>`s|`<Route><Route .../></Route>`|

---

✅ **In short:**  
`<Route>` defines _what to show_ for _which URL_. It’s always used inside `<Routes>`, and together they make React Router’s page mapping work.