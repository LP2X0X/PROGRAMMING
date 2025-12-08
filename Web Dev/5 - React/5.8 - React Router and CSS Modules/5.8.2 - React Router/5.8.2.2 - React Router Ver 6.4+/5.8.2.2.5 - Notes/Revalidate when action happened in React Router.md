---
tags: 
 - react
 - router
 - note
 - revalidate
---

**Any action submission) _can_ cause React Router to revalidate loaders, but only under specific rules.**

Here is the complete overview so you understand exactly when revalidation happens.

---

# **1. What happens when an action (POST) runs**

When you submit a form or fetcher to a route **action**, React Router will:

1. Run the action for that route.
    
2. Then decide whether to **revalidate loaders** based on its revalidation rules.
    

---

# **2. When React Router _will_ revalidate loaders**

React Router revalidates after an action **unless something stops it**.

Loaders revalidate when:

### **A. The action succeeds normally (default)**

If the action returns regular data:

```js
return { success: true };
```

React Router will re-run loaders for all relevant routes.

This is how the UI stays in sync after a mutation.

---

### **B. The action returns a redirect**

Redirects **always force revalidation** of the loaders on the destination route.

```js
return redirect("/dashboard");
```

---

### **C. The action uses `json()`**

Same as returning data — loaders revalidate afterward.

---

# **3. When React Router _does NOT_ revalidate**

### **A. If the action throws a redirect with `throw redirect()`**

This still navigates, but it **skips revalidation** of the routes before redirecting.

### **B. If the submission is `fetcher.submit()` and it targets `loader`**

(fetch requests with `method="get"` do not revalidate everything, only the target route)

### **C. If you explicitly disable revalidation**

Using `shouldRevalidate()` returning `false`.

---

# **4. POST via `<Form>` vs via `useFetcher`**

### **A. `<Form method="post">`**

Always triggers action → then revalidation of loaders.

### **B. `<fetcher.Form method="post">`**

Runs action → **then** revalidates loaders  
(unless `shouldRevalidate()` denies it)

So yes, fetcher POST still revalidates.

BUT fetcher GET does **not** revalidate other loaders.

---

# **5. In plain English**

A POST action is treated as a "mutation," so React Router assumes your data changed and re-runs loaders to refresh the UI.

That is the default behavior.

---

# **6. Summary Table**

| Submission Type                      | Action Runs? | Loader Revalidation?                          |
| ------------------------------------ | ------------ | --------------------------------------------- |
| `<Form method="post">`               | Yes          | Yes (default)                                 |
| `<Form method="get">`                | No           | Normal navigation — loaders for new route run |
| `fetcher.submit({ method: 'post' })` | Yes          | Yes (default)                                 |
| `fetcher.submit({ method: 'get' })`  | No           | No (only the target loader runs)              |
| `throw redirect()`                   | Yes          | No pre-redirect revalidation                  |
| `shouldRevalidate()`                 | Yes          | Only if it returns true                       |
