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
    