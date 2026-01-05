---
tags: 
 - web dev
---

In React Router, when a user submits a `<Form>` or uses a `fetcher`, the corresponding **action** receives a `Request` object.  
From that request, you can extract submitted form fields by calling:

```js
const formData = await request.formData();
```

This behaves the same as the **native `FormData` Web API**, but React Router handles the routing and submission for you.

---

# Where You Use It

### 1. **In an Action**

Actions run when a form is submitted (POST/PUT/PATCH/DELETE).

```jsx
export async function action({ request }) {
  const formData = await request.formData();

  const name = formData.get("name");
  const age = formData.get("age");
  
  return { name, age };
}
```

### 2. **In a Loader** (rare)

Only if you're handling GET form submissions (not typical).

---

# What `formData()` Returns

A `FormData` object containing all `<input>`, `<select>`, `<textarea>`, etc. fields with `name` attributes.

You can:

### Read a single value:

```js
formData.get("email");
```

### Read multiple values:

```js
formData.getAll("tags");
```

### Iterate:

```js
for (const [key, value] of formData) {
  console.log(key, value);
}
```

---

# Example: Complete Action With Validation

```jsx
export async function action({ request }) {
  const fd = await request.formData();

  const email = fd.get("email");
  const password = fd.get("password");

  if (!email || !password) {
    return { error: "Missing fields" };
  }

  // Do whatever logic you need

  return redirect("/dashboard");
}
```

---

# Example With `<Form>` Component

React Router automatically converts the HTML form into a fetch-and-route request.

```jsx
import { Form } from "react-router-dom";

export default function LoginForm() {
  return (
    <Form method="post">
      <input name="email" />
      <input name="password" type="password" />
      <button type="submit">Login</button>
    </Form>
  );
}
```

**Connected action:**

```js
export async function action({ request }) {
  const data = await request.formData();
  console.log(data.get("email"));
}
```

---

# How Files Work

React Router supports file uploads automatically if you use:

```html
<input type="file" name="avatar" />
```

Read it using:

```js
const file = formData.get("avatar"); // File object
```

---

# Relationship to `useActionData()`

Anything returned from the action can be read in the component:

```jsx
const errors = useActionData();
```

---

# Relationship to `useFetcher()`

Fetcher submissions also provide `request.formData()` in the fetcher’s action.

```jsx
const fetcher = useFetcher();

<fetcher.Form method="post">
  <input name="query" />
</fetcher.Form>
```

Action:

```js
export async function action({ request }) {
  const fd = await request.formData();
  return fd.get("query");
}
```

---

# Key Points Summary

- `request.formData()` is the **official way to read submitted form values** in React Router actions.
    
- Returns a **FormData** object (native API).
    
- Works with `<Form>` and `<fetcher.Form>`.
    
- Only `name`d fields are included.
    
- Supports file uploads.
    
- Often used together with `useActionData`, `redirect`, or return JSON.
    