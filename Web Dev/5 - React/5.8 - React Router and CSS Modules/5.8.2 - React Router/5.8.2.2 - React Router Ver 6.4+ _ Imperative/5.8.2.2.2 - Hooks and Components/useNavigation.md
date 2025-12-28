---
tags: 
 - react
 - router
 - hook
---

```ad-note
Not to be confused with [[Programmatic Navigation|useNavigate]].
```

`useNavigation()` is a React Router hook that lets you **detect and track ongoing navigations**, especially during **loader/actions** in data-router apps.

It is basically **React Router’s built-in "global loading state"**, or in other words, it is universal for the entire application.

---

## ✅ **What it returns**

`useNavigation()` returns an object describing the current navigation:

```js
const navigation = useNavigation();
```

The shape:

```ts
{
  state: "idle" | "submitting" | "loading",
  location: Location | null,
  formData: FormData | null,
  formMethod: "get" | "post" | null
}
```

---

## 🚦 **Understanding `state`**

### `state: "idle"`

- Nothing is happening.
    
- No route is loading.
    
- No form submission in progress.
    

### `state: "submitting"`

- You submitted a `<Form>` or used `fetcher.submit()`.
    
- Data has been sent to the server-side `action`, but no response yet.
    

### `state: "loading"`

- A navigation or loader is currently fetching data.
    
- Happens after an action returns and route reload begins.
    
- Also happens for simple link navigations when loader runs.
    

---

## 🎯 **What it's used for**

### 1️⃣ **Global loading UI**

```jsx
const navigation = useNavigation();

return (
  <>
    {navigation.state !== "idle" && <LoadingSpinner />}
    <Outlet />
  </>
);
```

### 2️⃣ **Disable buttons while submitting**

```jsx
<button disabled={navigation.state === "submitting"}>
  {navigation.state === "submitting" ? "Saving..." : "Save"}
</button>
```

### 3️⃣ **Show skeleton when navigating to a new page**

```jsx
if (navigation.state === "loading") {
  return <PageSkeleton />;
}
return <ActualPage />;
```

### 4️⃣ **Check where the user is going**

```jsx
useEffect(() => {
  if (navigation.location) {
    console.log("Navigating to:", navigation.location.pathname);
  }
}, [navigation.location]);
```

---

## 📝 **Practical Summary**

- `useNavigation()` tracks **navigation, loader fetches, and action submissions**.
    
- Replaces the need for manual “isLoading” state.
    
- Works automatically with `<Form>`, `action()`, `loader()`, and router navigations.
    
- Useful for **global or per-page loading indicators**.
    

---

## 🔥 Short Remember-Note

> **`useNavigation()` tells you what React Router is doing right now:  
> idle, submitting, or loading.  
> Use it to show spinners, disable UI, or display skeletons.**