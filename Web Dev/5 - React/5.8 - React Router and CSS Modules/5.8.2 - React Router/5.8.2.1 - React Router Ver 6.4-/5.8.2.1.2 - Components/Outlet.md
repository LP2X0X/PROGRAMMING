---
tags: 
 - react
 - router 
 - component
 - nested
---

### 🧩 **`<Outlet>` Overview**

**Purpose:**  
A placeholder that renders the **child route’s component** inside a **parent route’s layout**.

---

### 🧱 **Basic Syntax**

```jsx
import { Outlet } from "react-router-dom";

function Layout() {
  return (
    <div>
      <Header />
      <Outlet /> {/* Child route renders here */}
      <Footer />
    </div>
  );
}
```

---

### ⚙️ **Used With Nested Routes**

```jsx
<Routes>
  <Route path="/dashboard" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="settings" element={<Settings />} />
  </Route>
</Routes>
```

- `/dashboard` → shows `Layout + Home`
    
- `/dashboard/settings` → shows `Layout + Settings`
    

---

### 🧠 **Key Points**

|Feature|Description|
|---|---|
|Acts as a **render slot**|Displays the matched child route|
|Used **only in parent routes**|Child components render inside it|
|Keeps layout persistent|Header/nav stay the same between child pages|

---

✅ **In short:**  
`<Outlet>` is where the **nested route’s content appears** inside a parent component’s layout.