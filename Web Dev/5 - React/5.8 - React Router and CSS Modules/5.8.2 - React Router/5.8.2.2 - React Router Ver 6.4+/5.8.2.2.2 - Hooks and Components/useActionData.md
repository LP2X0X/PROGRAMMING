---
tags: 
 - react
 - router
 - hook
---

# ⚡ `useActionData()` — Detailed Overview

`useActionData()` is a React Router hook that lets your component **access the data returned from an `action()` function** after a form submission.

It is ONLY for forms that use:

- `<Form method="post">`
    
- `fetcher.submit()`
    
- actions in route config
    

---

# 📌 1. How it works (mental model)

When you submit a form:

1. React Router runs the route’s `action()`
    
2. Your `action()` returns some data (usually validation errors)
    
3. `useActionData()` lets the component read that returned data
    

Example:

```jsx
const data = useActionData();
```

`data` contains **whatever the action returned**.

---

# 📌 2. Basic Example (most common use case: form errors)

### Component:

```jsx
import { Form, useActionData } from "react-router-dom";

function LoginForm() {
  const errors = useActionData();

  return (
    <Form method="post">
      <input type="text" name="email" />
      {errors?.email && <p>{errors.email}</p>}

      <input type="password" name="password" />
      {errors?.password && <p>{errors.password}</p>}

      <button>Login</button>
    </Form>
  );
}
```

### Action:

```js
export async function action({ request }) {
  const formData = await request.formData();
  const email = formData.get("email");
  const password = formData.get("password");

  const errors = {};
  if (!email) errors.email = "Email is required";
  if (!password) errors.password = "Password is required";

  if (Object.keys(errors).length > 0) {
    return errors; // <-- returned to useActionData()
  }

  // success case
  return redirect("/dashboard");
}
```

---

# 📌 3. When does `useActionData()` update?

It runs **ONLY after an action submission on the same route**.

- Submitting a form → action runs → hook updates
    
- Clicking links → loader runs → hook **does NOT update**
    
- Navigating to another page → hook resets to `undefined`
    

---

# 📌 4. Relationship: `useActionData()` vs `useNavigation()`

### `useActionData()`

- Only gives you **action return value**
    
- Used for validation errors, server responses
    

### `useNavigation()`

- Tells you about loading/submitting state
    
- Used for spinners, disabling UI
    

Example:

```jsx
const navigation = useNavigation();
const data = useActionData();

if (navigation.state === "submitting") {
  return <p>Submitting...</p>;
}
```

---

# 📌 5. Common Patterns

### ✔ Form validation

```js
if (!email) return { error: "Email required" };
return redirect("/success");
```

### ✔ Return new created data

```js
return { id: newUser.id };
```

### ✔ Server-side error

```js
return { message: "Wrong password" };
```

### ✔ Re-render form with failed submission data

```jsx
<input name="email" defaultValue={data?.email} />
```

---

# 📌 6. Rules / Restrictions

❌ Cannot be used in loaders  
❌ Only updates after submissions on the same route  
❌ Does not persist across navigation  
❌ Does not get triggered on GET requests (link clicks)

---

# 📌 7. Short Remember-Note

> **`useActionData()` gives you whatever the action returns.  
> Use it for form errors, server-provided messages, or failed validations.**