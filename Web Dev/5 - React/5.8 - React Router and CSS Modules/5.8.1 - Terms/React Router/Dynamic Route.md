---
tags: 
 - react
 - route
 - term
---

In **React**, a _dynamic route_ lets you create pages or components whose content depends on **a changing part of the URL** — for example, `/users/123` or `/products/iphone-15`.

How you set this up depends on what routing library you’re using — typically **React Router (v6+)** or **Next.js**.  
Here’s how to do it in both 👇

---

## 🧭 1. Dynamic Routes with **React Router v6+**

### ✅ Step-by-step example

**Install React Router**

```bash
npm install react-router-dom
```

**Setup in `App.jsx`**

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import UserDetail from "./pages/UserDetail";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        {/* 👇 Dynamic route */}
        <Route path="/users/:userId" element={<UserDetail />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

**Then in `UserDetail.jsx`**

```jsx
import { useParams } from "react-router-dom";

function UserDetail() {
  const { userId } = useParams(); // get "123" from /users/123
  return <h2>User ID: {userId}</h2>;
}

export default UserDetail;
```

➡️ Visiting `/users/123` will render `UserDetail` showing `User ID: 123`.

---

## ⚡ 2. Dynamic Routes with **Next.js**

In Next.js, dynamic routing is **file-based**.  
You define the route using square brackets in the filename.

### Example: `/pages/users/[id].js`

```jsx
import { useRouter } from "next/router";

export default function UserPage() {
  const router = useRouter();
  const { id } = router.query; // same as useParams
  return <h1>User ID: {id}</h1>;
}
```

✅ Works automatically — no manual route configuration needed.  
Visiting `/users/123` → renders this page with `id = 123`.

---

## 🧩 You can also nest dynamic routes

**React Router**

```jsx
<Route path="/products/:productId/reviews/:reviewId" element={<Review />} />
```

**Next.js**

```
pages/products/[productId]/reviews/[reviewId].js
```