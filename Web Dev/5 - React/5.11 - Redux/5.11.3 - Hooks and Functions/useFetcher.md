---
tags: 
 - react 
 - router
 - hook
---

# 🎯 **useFetcher — Detailed Note (React Router v6.4+)**

`useFetcher()` is a special hook from **React Router’s Data APIs** that lets your component:

- submit forms or actions
    
- load data
    
- manage pending / loading / submitting states
    
- **without navigating** to another route
    

It gives you a mini “request machine” that works like `fetch()` but is fully integrated with React Router’s loaders and actions.

---

# 📌 **1. What is useFetcher()?**

```js
const fetcher = useFetcher();
```

It returns an object that lets you:

### ▶ Submit data to an action (POST, PUT, DELETE…)

Without navigating.

### ▶ Load data through a loader

Without navigating.

### ▶ Track state: "idle" | "submitting" | "loading"

Like built-in fetch UI state.

### ▶ Render the results provided by that loader/action

Via `fetcher.data`

### ▶ Render a form that submits through the router

`<fetcher.Form method="post">`

---

# 📦 **2. What does useFetcher() return?**

```js
const {
  Form,        // fetcher-aware <form> component
  submit,      // programmatic submission
  load,        // programmatic loader call
  state,       // "idle" | "loading" | "submitting"
  data,        // resolved data from loader/action
  formData,    // FormData being submitted
  method,      // "get" | "post"
} = useFetcher();
```

---

# 🔄 **3. When to use useFetcher instead of normal navigation?**

Use it when you want:

### ✔ Calling a loader _without navigation_

E.g., live search suggestions.

### ✔ Submitting a form _without navigation_

E.g., like/unlike button, add to cart.

### ✔ Run background requests

E.g., refetch current route’s data after mutation.

### ✔ Submit data _from a component located anywhere_

not tied to the route where the action/loader lives.

---

# 🧭 **4. Example: Mutating Data Without Navigating**

```jsx
function LikeButton({ postId }) {
  const fetcher = useFetcher();

  return (
    <fetcher.Form method="post" action={`/posts/${postId}/like`}>
      <button disabled={fetcher.state === "submitting"}>
        {fetcher.state === "submitting" ? "Liking..." : "Like"}
      </button>
    </fetcher.Form>
  );
}
```

🔹 Clicking the button calls the action  
🔹 Page does NOT navigate  
🔹 You get pending state from `fetcher.state`

---

# 🔍 **5. Example: Loading Data Without Navigation**

```jsx
function LiveSearch() {
  const fetcher = useFetcher();

  function handleChange(e) {
    fetcher.load(`/search?query=${e.target.value}`);
  }

  return (
    <div>
      <input onChange={handleChange} />
      {fetcher.state === "loading" && <p>Searching...</p>}
      {fetcher.data && fetcher.data.results?.map(r => <div>{r}</div>)}
    </div>
  );
}
```

You call a loader from _another route_ and show results inline.

---

# 🧩 **6. Internal Behavior — How useFetcher works**

When you call:

### `fetcher.submit(formData, options)`

or

### `<fetcher.Form />`

or

### `fetcher.load(url)`

React Router does **not** change your location.  
Instead it:

1. Looks up the correct loader/action for that URL
    
2. Calls the loader/action
    
3. Tracks `state`:
    
    - `idle → submitting → loading → idle`
        
4. Stores the data in the fetcher instance
    
5. Re-renders your component with the new `fetcher.data`
    

Important point:

- Each call creates a **unique key**
    
- A component can use _multiple_ simultaneous fetchers
    

---

# 🏷 **7. States**

```
idle → submitting → loading → idle
```

Where:

- **submitting** = form data is being sent
    
- **loading** = waiting for loader to resolve after submission
    
- **idle** = no active request
    

---

# 🛠 **8. Programmatic Submission**

```js
fetcher.submit(
  { title: "Hello" },
  { method: "post", action: "/posts/create" }
);
```

Equivalent to submitting a form.

---

# 📌 **9. Data Lifetimes**

`fetcher.data` stays available until:

- You make a new request
    
- Component unmounts
    
- You manually clear it (by new load/submit)
    

Fetcher does NOT automatically reset after a delay.

---

# 🎛 **10. Why use useFetcher instead of fetch()?**

|fetch()|useFetcher()|
|---|---|
|manual state management|automatic pending/loading/submitting states|
|not tied to router|integrated with router loaders/actions|
|doesn’t integrate with router cache|uses loader/action pipeline|
|cannot submit to router actions|can submit directly to actions|
|no automatic revalidation|router revalidation system works|

---

# 🧠 **Mental Model**

> `useFetcher()` gives you a tiny client-side “fetch machine” that can talk to any loader/action in the app — without changing the URL — and gives you built-in UI state for the request.