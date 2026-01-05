[](Submit%20form%20without%20React%20Router.md)[](action.md)[](Submit%20form%20without%20React%20Router.md)[](action.md)---
tags: 
 - react
 - router 
 - hook
---

# 🚀 **`<Form>` in React Router — Full Overview**

`<Form>` is a React Router component that behaves like a normal HTML `<form>`, but it automatically submits data to a **route action()** instead of doing a full page reload.

It is part of the Data APIs that include:

- loaders
    
- actions
    
- useNavigation
    
- useLoaderData
    
- useRouteError
    

---

# 🎯 **Why use `<Form>` instead of `<form>`?**

### ✔ 1. It submits without reloading the page

React Router intercepts the submit and calls the route’s `action()`.

### ✔ 2. It automatically serializes form data

You don’t need `onSubmit`, `preventDefault`, or manual fetch.

### ✔ 3. It integrates with data mutations

After an action runs, React Router will:

- automatically revalidate loaders
    
- update the data on the page
    
- handle redirects correctly
    
- show pending UI states
    

### ✔ 4. It gives you navigation state

You can detect:

- submitting
    
- loading
    
- idle
    

Using `useNavigation()`.

---

# 📌 **Basic example**

## Route config:

```jsx
const router = createBrowserRouter([
  {
    path: "/contact/:id",
    element: <Contact />,
    action: updateContactAction,
  },
]);
```

## Component with `<Form>`:

```jsx
import { Form, useNavigation } from "react-router-dom";

function Contact() {
  const navigation = useNavigation();
  const isSubmitting = navigation.state === "submitting";

  return (
    <Form method="post">
      <input name="firstName" />
      <input name="lastName" />

      <button type="submit">
        {isSubmitting ? "Saving..." : "Save"}
      </button>
    </Form>
  );
}
```

## Action:

```ts
export async function updateContactAction({ request, params }) {
  const formData = await request.formData();
  const firstName = formData.get("firstName");
  const lastName = formData.get("lastName");

  // Update DB, call API, etc...
  await updateUser(params.id, { firstName, lastName });

  return redirect(`/contact/${params.id}`);
}
```

---

# 🧩 **How it works inside**

When you click submit:

1. `<Form>` intercepts the event
    
2. `react-router` creates a `Request` object
    
3. It calls the route’s `action({ request })`
    
4. The action returns:
    
    - `data` → stays on the same page
        
    - `redirect()` → auto redirects
        
    - `throw new Response()` → error boundary
        
5. All loaders that "depend" on this route are revalidated
    
6. UI updates automatically
    

---

# ⚙️ **Props**

### `method`

The HTTP verb to use. Supports `get`, `post`, `put`, `patch`, `delete`.

```jsx
<Form method="post">
<Form method="get">
<Form method="put">
<Form method="delete">
```

Yes — React Router supports non-GET/POST methods using `fetcher` or by overriding the method using a hidden field.

### `action`

The URL where the form data will be submitted. Defaults to the closest matching route's URL.

```jsx
<Form method="post" action="/users/create">
```

### `replace`

If `true`, the navigation replaces the current entry in the history stack instead of pushing a new one.

```jsx
<Form replace>
```

### `reloadDocument`

If `true`, forces a full document navigation (like a standard HTML form submit) instead of client-side routing.

---

# 👉 **Handling validation errors**

Return error data from the action:

```ts
export async function action({ request }) {
  const data = await request.formData();
  if (!data.get("email")) {
    return { error: "Email is required" };
  }
}
```

Use it inside the component:

```jsx
import { useActionData } from "react-router-dom";

function MyForm() {
  const data = useActionData();

  return (
    <Form method="post">
      {data?.error && <p>{data.error}</p>}
      <input name="email" />
      <button>Submit</button>
    </Form>
  );
}
```

---

# 🔄 **Show pending UI**

```jsx
const navigation = useNavigation();

if (navigation.state === "submitting") {
  return <p>Submitting…</p>;
}
```

States:

- `"idle"`
    
- `"submitting"`
    
- `"loading"`
    

---

# 🎒 **Example: Delete button**

A very common use case:

```jsx
<Form method="post">
  <button name="_action" value="delete">Delete</button>
</Form>
```

Inside the action:

```ts
const formData = await request.formData();
if (formData.get("_action") === "delete") {
  await deleteContact(params.id);
  return redirect("/contacts");
}
```

---

# 🔥 Quick Summary (Cheat sheet)

### `<Form>`:

- behaves like HTML form
    
- but sends data to route `action()`
    
- no page reload
    
- auto revalidate loaders
    
- integrates with navigation state
    
- extremely clean way to mutate data
    