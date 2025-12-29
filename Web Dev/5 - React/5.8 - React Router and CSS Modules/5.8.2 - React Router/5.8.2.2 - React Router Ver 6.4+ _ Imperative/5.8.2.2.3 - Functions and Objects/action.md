---
tags: 
 - react
 - router
 - function
---

## **1. What is an `action` function?**

An **`action` function** is a special function you can attach to a route that handles **form submissions** or any other **POST-like interactions**.  
It runs **on the server or client**, depending on how your app is deployed, when a form with `method="post"` (or other HTTP methods) is submitted.

- Think of it as the **handler for the route when data is sent**.
    
- Similar to `onSubmit` but declaratively tied to a route.
    

---

## **2. Where to define it**

```ad-tip
You should define it inside the component file which the route render then pass to the action property of the route object.
```

```javascript
import { createBrowserRouter } from "react-router-dom";
import MyFormPage from "./MyFormPage";

const router = createBrowserRouter([
  {
    path: "/form",
    element: <MyFormPage />,
    action: async ({ request, params }) => {
      // handle form submission here
    },
  },
]);
```

- Can be **async** to handle API calls or server processing.
    

---

## **3. Function signature**

```ts
type ActionFunction = (args: {
  request: Request;
  params: Record<string, string>;
}) => any;
```

- **`request`**: The native [Request object](https://developer.mozilla.org/en-US/docs/Web/API/Request)
    
    - You can call `await request.formData()` to get form fields.
        
    - Check `request.method` to see if it’s POST, PUT, DELETE, etc.
        
- **`params`**: Route params, e.g., `/users/:id` → `{ id: "123" }`.
    

````ad-note
You must return something from the action function or you will encounter this error:
```ad-error
You defined an action for route "routes/" but didn't return anything from your action function. Please return a value or null.
```
````

---

## **4. Example usage**

**Form component:**

```jsx
import { Form } from "react-router-dom";

export default function MyFormPage() {
  return (
    <Form method="post">
      <input name="username" placeholder="Username" />
      <button type="submit">Submit</button>
    </Form>
  );
}
```

**Action function:**

```javascript
export async function action({ request }) {
  const formData = await request.formData();
  const username = formData.get("username");

  // e.g., save to server or call API
  console.log("Form submitted:", username);

  // return redirect to another page
  return redirect("/thank-you");
}
```

- Using `Form` from React Router automatically **calls the `action` of the route**.
    
- You don’t need an `onSubmit` handler in the component itself.
    
- You can return:
    
    - A redirect (`redirect("/path")`)
        
    - Data (`return { success: true }`) that can be used in `useActionData()` hook.
        

```ad-note
You could return an object represent error and the form component can access and handle that error through [[useActionData]] hook.
```

---

## **5. Related hooks**

- **`useActionData()`** – get the data returned from your `action`.
    
- **`useNavigation()`** – check form submission/loading state.
    
- **`useSubmit()`** – programmatically submit forms to a route’s `action`.
    

---

## **6. Notes / Best Practices**

1. **`action` is route-specific**
    
    - Each route can have its own `action`.
        
2. **Server-like behavior**
    
    - It’s ideal for handling server calls, mutations, or form submissions.
        
3. **No component state needed for submission**
    
    - The logic lives in the `action`, making the UI simpler.
        
4. **Supports `async/await`** naturally.
    
5. **Works with `redirect`** to navigate after submission without manual navigation.
    

---

💡 **TL;DR:**

- `action` handles **form submission and mutations** for a route.
    
- You get `request` and `params`.
    
- Use it with `<Form>` or `useSubmit()` to do server-like handling declaratively.
    

----

Below is a **clear, complete, and accurate overview** of _why an action must always return a value_ in React Router, and _what types of values you are allowed to return_.

---

# 1. Why an action must always return something

React Router treats **actions as server-like request handlers**.  
Every POST/PUT/PATCH/DELETE form submission or fetcher submission must produce a response.

An **action must always return** because React Router uses the return value to determine what to do next:

1. **Populate `fetcher.data` or `useActionData()`**
    
2. **Trigger revalidation** (re-run affected loaders)
    
3. **Detect redirects**
    
4. **Handle errors**
    
5. **Complete the navigation or fetcher request**
    

If you return nothing (or forget to return):

```js
async function action() {
  doSomething();   // NO return
}
```

React Router treats it as returning **undefined**, which is allowed but often unintended — and will still revalidate and finish the request.

But:

- you _cannot_ simply omit `return` if you intend to redirect
    
- and you _cannot_ send custom data unless you return it
    

So:  
> **Action must return because the router needs a response for the request life cycle.**

---

# 2. What can an action return?

An action can return any of the following:

## A. **Redirect**

The most common use.

```js
import { redirect } from "react-router";

export async function action({ request }) {
  return redirect("/dashboard");
}
```

This tells React Router to stop and navigate immediately.

---

## B. **Any JSON-serializable data**

```js
import { json } from "react-router";

export async function action({ request }) {
  const formData = await request.formData();
  const message = formData.get("message");

  return json({ success: true, message });
}
```

This becomes available through:

- `useActionData()` for normal forms
    
- `fetcher.data` for fetcher forms
    

---

## C. **A primitive value (string, number, boolean, null)**

React Router allows returning plain JS values:

```js
return "OK";
return 42;
return false;
return null;
return undefined;
```

But usually you prefer `json()` for clarity.

---

## D. **A Response object (native Web Fetch API)**

You can return a full custom response:

```js
return new Response("Custom body", {
  status: 201,
  headers: { "Content-Type": "text/plain" }
});
```

This gives total manual control.

---

## E. **Throw errors**

Either manually:

```js
throw new Error("Something went wrong");
```

Or using `json` with error status:

```js
throw json({ error: "Invalid form" }, { status: 400 });
```

React Router catches it and routes it to the nearest Error Boundary.

---

## F. **Nothing (undefined)** — allowed but limited

```js
return;
```

This is the same as `return undefined`.

React Router will:

- return `undefined` to the fetcher or `useActionData()`
    
- still revalidate loaders
    

This is valid, but rarely intentional.

---

# 3. Summary Table

|Return Type|Purpose|Notes|
|---|---|---|
|`redirect("/path")`|Navigate immediately|Most common for form actions|
|`json({...})`|Return structured data|Recommended for fetchers|
|Primitive values|Simple responses|Still passed through to fetcher/actionData|
|`Response`|Full control|You manage headers & status|
|`throw ...`|Error handling|Routed to errorElement|
|`undefined`|“No data”|Revalidation still happens|

---

# 4. Why React Router enforces the return requirement

Because **actions simulate server endpoints**.

A server endpoint must always produce a response:

- 200 OK
    
- Redirect
    
- 404
    
- Error
    
- etc.
    

React Router implements the same life cycle.