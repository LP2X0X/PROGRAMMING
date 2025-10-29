---
tags: 
 - react
 - router
 - navigation
 - technique
---

Programmatic navigation in **React Router** means navigating the user to a different route **using code**, not links (`<Link>` or `<NavLink>`).

Here’s a complete breakdown depending on your React Router version 👇

---

## ⚙️ React Router v6 (the current major version)

You use the **`useNavigate()`** hook.

---

### 🧭 1️⃣ Basic Example

```jsx
import { useNavigate } from "react-router-dom";

function Home() {
  const navigate = useNavigate();

  const goToAbout = () => {
    navigate("/about");
  };

  return (
    <button onClick={goToAbout}>Go to About Page</button>
  );
}
```

✅ `navigate("/about")` changes the URL and renders the `<About />` route.  
It’s the programmatic equivalent of clicking a `<Link to="/about" />`.

---

### 🧱 2️⃣ Navigate with Parameters or Query Strings

```jsx
navigate(`/user/${userId}`);
navigate(`/search?query=${searchTerm}`);
```

If you’re using **search params**, React Router also provides:

```jsx
import { useSearchParams } from "react-router-dom";

const [searchParams, setSearchParams] = useSearchParams();
setSearchParams({ q: "react", page: 2 });
```

---

### ♻️ 3️⃣ Navigate Back or Forward

```jsx
navigate(-1); // go back
navigate(1);  // go forward
```

This mimics browser history navigation.

---

### ⚙️ 4️⃣ Replace the Current Entry in History

By default, `navigate()` **pushes** a new entry to history (like `pushState`).

You can replace it instead (like `replaceState`):

```jsx
navigate("/login", { replace: true });
```

Useful when redirecting after logout or form submission — you don’t want the user to go “back” to the previous page.

---

### 🔐 5️⃣ Conditional Navigation (Redirect after action)

```jsx
function Login() {
  const navigate = useNavigate();

  const handleLogin = () => {
    // simulate successful login
    localStorage.setItem("auth", "true");
    navigate("/dashboard");
  };

  return <button onClick={handleLogin}>Login</button>;
}
```

---

### ⏩ 6️⃣ Redirect on Mount (auto redirect)

```jsx
import { useEffect } from "react";
import { useNavigate } from "react-router-dom";

function ProtectedRoute() {
  const navigate = useNavigate();
  const isLoggedIn = false;

  useEffect(() => {
    if (!isLoggedIn) navigate("/login");
  }, [isLoggedIn, navigate]);

  return <div>Secret content</div>;
}
```

---

### 🧠 7️⃣ Inside Class Components (React Router v6+)

Class components don’t support hooks directly, but you can use a **wrapper**:

```jsx
import { useNavigate } from "react-router-dom";

function withNavigation(Component) {
  return (props) => <Component {...props} navigate={useNavigate()} />;
}
```

Then in your class component:

```jsx
class Profile extends React.Component {
  handleLogout = () => {
    this.props.navigate("/login");
  };

  render() {
    return <button onClick={this.handleLogout}>Logout</button>;
  }
}

export default withNavigation(Profile);
```

---

### ⚡ Bonus: `Navigate` Component (for declarative redirects)

If you need to redirect in JSX (not in code):

```jsx
import { Navigate } from "react-router-dom";

function ProtectedRoute() {
  const isAuth = false;
  return isAuth ? <Dashboard /> : <Navigate to="/login" replace />;
}
```

---

## 📚 Summary Table

|Task|Method|Example|
|---|---|---|
|Go to route|`navigate("/path")`|`navigate("/about")`|
|Back / forward|`navigate(-1)` / `navigate(1)`|`navigate(-1)`|
|Replace current|`navigate("/login", { replace: true })`|–|
|With params|template string|``navigate(`/user/${id}`)``|
|In JSX|`<Navigate to="/path" />`|`<Navigate to="/login" />`|