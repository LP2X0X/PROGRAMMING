---
tags: 
 - react
 - router
 - technique
---

### Prerequisite
[[action]]

---

```ad-question

**How do we get component state**—like stuff from a fancy rich text editor, dynamically generated arrays, or any data not sitting in a standard `<input name="...">`—**into React Router's `action` function?**
```

---

# ✅ Answer

A **hidden input** is a normal HTML input that is invisible but still submitted with the form.

```html
<input type="hidden" name="something" value="secret" />
```

React Router’s `<Form>` behaves exactly like an HTML `<form>`, so hidden inputs work perfectly.

---

## 📌 **Why use hidden inputs?**

Hidden inputs are perfect when you want to send:

✔ A record ID  
✔ Mode (create/edit/delete)  
✔ CSRF token  
✔ Flags for the `action`  
✔ Values that the user shouldn’t change  
✔ Values that don’t belong in visible UI

---

## 📘 **Example: Send extra data to an action**

#### **Component**

- Refer to [[POST Request Pitfalls#**2. Not Stringifying the Body**|this]] for why we need to stringify state variable.
```jsx
import { Form } from "react-router-dom";

export default function DeleteUser({ userId }) {
  // package will look like this {order: car}
  // it can be a more complex and nested array of objects
  const [package, setPackage] = useState();
  return (
    <Form method="post">
      <input type="hidden" name="package" value={JSON.stringify(package)} />
      <button type="submit">Do something</button>
    </Form>
  );
}
```

---

## 📘 **In the action: Read the hidden values**

```js
export async function action({ request }) {
  const formData = await request.formData();
  const package = formData.get("package");

  return null;
  }
}
```

---

## ⭐ Why this technique is great

#### ✔ 1. Lets one `<Form>` perform multiple behaviors

(You can switch behaviors by sending different “intent” values.)

#### ✔ 2. No need for extra routes or multiple forms

#### ✔ 3. Solves the “which button was pressed?” problem

(Hidden input can be set dynamically based on which button is clicked.)

---

## 📌 **Dynamic hidden inputs (common pattern)**

```jsx
<Form method="post">
  <input type="hidden" name="intent" id="intent-input" />

  <button
    type="submit"
    onClick={() => {
      document.getElementById("intent-input").value = "approve";
    }}
  >
    Approve
  </button>

  <button
    type="submit"
    onClick={() => {
      document.getElementById("intent-input").value = "reject";
    }}
  >
    Reject
  </button>
</Form>
```

#### In the `action`:

```js
const intent = formData.get("intent"); // "approve" or "reject"
```

---

## 📌 **Hidden input with fetcher (AJAX style)**

```jsx
const fetcher = useFetcher();

<fetcher.Form method="post">
  <input type="hidden" name="intent" value="toggle-done" />
  <input type="hidden" name="todoId" value={todo.id} />
  <button>✓</button>
</fetcher.Form>
```

---

## 📌 Summary

Hidden inputs are a **core technique** in React Router `action` functions to pass extra metadata into the server-side handlers.

|Purpose|Hidden Input Example|
|---|---|
|Identify operation|`<input name="intent" value="delete" />`|
|Pass IDs|`<input name="id" value={id} />`|
|Send flags|`<input name="from" value="profile" />`|
|Switch behavior inside action|`intent = formData.get("intent")`|