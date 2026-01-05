---
tags: 
 - react
 - query
 - custom
 - hook
---

## 🧠 What Is a `useQuery` Custom Hook?

A **`useQuery` custom hook** is a **domain-specific abstraction** built on top of React Query that:

- Encapsulates **query logic**
    
- Centralizes **query keys**
    
- Standardizes **fetching behavior**
    
- Improves **reuse, consistency, and readability**
    

Instead of calling `useQuery` directly in every component, you create a **semantic hook**.

---

## 🎯 Why Create a Custom Hook?

### 📌 Separation of Concerns

Components focus on **UI**, not data-fetching mechanics.

### 📌 Single Source of Truth

- One query key
    
- One fetch function
    
- One configuration
    

### 📌 Safer Refactoring

Changing:

- API endpoints
    
- Query options
    
- Caching behavior
    

…only requires updating **one place**.

---

## 🧩 Basic Pattern

### 🔹 Raw `useQuery` (not ideal)

```ts
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
});
```

### 🔹 Custom Hook (recommended)

```ts
function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
}
```

Now your component reads declaratively:

```ts
const { data, isLoading } = useUsers();
```

---

## 🏗️ Typical Custom Hook Structure

### 🧱 Structure Breakdown

```ts
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000,
  });
}
```

|Part|Purpose|
|---|---|
|Hook name|Expresses intent|
|`queryKey`|Cache identity|
|`queryFn`|Data source|
|Options|Behavior control|

---

## 🧠 Query Key Design (Critical)

### 🔑 Static Query

```ts
['users']
```

### 🔑 Dynamic Query

```ts
function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
}
```

**Rule:**

> All inputs that affect the query **must be in the query key**

---

## 🔁 Parameterized Custom Hooks

Custom hooks can accept arguments cleanly:

```ts
function usePostsByUser(userId: string) {
  return useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId),
    enabled: Boolean(userId),
  });
}
```

This keeps components simple and declarative.

---

## ⚙️ Encapsulating Options

Custom hooks are ideal for **policy decisions**:

```ts
function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
    staleTime: 10 * 60 * 1000,
    retry: 2,
    refetchOnWindowFocus: false,
  });
}
```

Now **all consumers share the same caching rules**.

---

## 🧠 Returning Only What You Want (Optional Pattern)

You may **reshape the return value**:

```ts
function useUsers() {
  const query = useQuery({ ... });

  return {
    users: query.data,
    isLoading: query.isLoading,
    isError: query.isError,
  };
}
```

This creates a **cleaner API** for components.

---

## 🚫 What NOT to Do

### ❌ Do Not Fetch Inside the Hook Manually

```ts
// ❌ Wrong
useEffect(() => {
  fetch(...)
}, []);
```

React Query already manages:

- Lifecycle
    
- Caching
    
- Deduplication
    
- Refetching
    

---

### ❌ Do Not Hide Errors Silently

Avoid swallowing query state:

```ts
// ❌ Loses control
return query.data;
```

Always expose loading/error state unless intentional.

---

## 🧠 Relationship to Declarative Data Fetching

A `useQuery` custom hook is:

- **Declarative**: describes _what data you need_
    
- **Idempotent**: safe to call repeatedly
    
- **Cache-aware**: no duplicate requests
    

This aligns with **modern React Router loaders**, **Suspense**, and **server-state patterns**.

---

## 🧠 When to Create a Custom Hook?

Create one when:

- The query is used in **more than one place**
    
- The query has **non-trivial options**
    
- The data represents a **domain concept** (users, orders, posts)
    

Avoid one only if the query is **truly one-off**.

---

## 🧠 One-Sentence Summary

> A `useQuery` custom hook turns low-level data fetching into a **reusable, declarative domain API** that centralizes caching, behavior, and intent.