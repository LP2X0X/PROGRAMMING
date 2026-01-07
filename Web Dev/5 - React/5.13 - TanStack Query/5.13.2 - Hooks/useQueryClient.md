---
tags: 
 - react
 - query
 - custom
 - hook
---

## 🧠 What `useQueryClient` Is

`useQueryClient` is a hook that gives you **direct access to the Query Client instance**.

```ts
const queryClient = useQueryClient();
```

The **Query Client** is the **central cache manager** of React Query.

Think of it as:

> “The database and controller for all queries and mutations in your app.”

---

## 🧩 Why `useQueryClient` Exists

Hooks like `useQuery` and `useMutation`:

- Manage **one query or mutation**
    
- Are scoped to a component
    

`useQueryClient` lets you:

- Coordinate **multiple queries**
    
- Update cache **imperatively**
    
- React to mutations globally
    

---

## 🏗️ Where the Query Client Comes From

At app startup:

```tsx
const queryClient = new QueryClient();

<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

`useQueryClient()` retrieves **this same instance** from React context.

---

## 🔑 Core Responsibilities of the Query Client

The Query Client:

- Stores query cache
    
- Tracks query states
    
- Deduplicates requests
    
- Triggers re-renders
    
- Handles invalidation and refetching
    

`useQueryClient` is your **control panel** for these responsibilities.

---

## 🔄 Most Common Use Case: Cache Invalidation

After a mutation:

```ts
const queryClient = useQueryClient();

queryClient.invalidateQueries({
  queryKey: ["cabins"],
});
```

### What Happens Internally

1. Marks matching queries as **stale**
    
2. Triggers refetch if they are active
    
3. UI updates automatically
    

This is why React Query feels “automatic”.

---

## ✍️ Direct Cache Updates (`setQueryData`)

Instead of refetching:

```ts
queryClient.setQueryData(["cabins"], (old) =>
  old.filter((c) => c.id !== id)
);
```

Use cases:

- Optimistic UI
    
- Fast updates
    
- Avoid unnecessary network requests
    

---

## 🔍 Reading from the Cache (`getQueryData`)

```ts
const cabins = queryClient.getQueryData(["cabins"]);
```

Used for:

- Optimistic updates
    
- Conditional logic
    
- Rollbacks
    

---

## 🧪 Optimistic Updates Workflow

```ts
onMutate: async (id) => {
  await queryClient.cancelQueries(["cabins"]);

  const previous = queryClient.getQueryData(["cabins"]);

  queryClient.setQueryData(["cabins"], (old) =>
    old.filter((c) => c.id !== id)
  );

  return { previous };
}
```

Rollback on error:

```ts
onError: (_err, _vars, ctx) => {
  queryClient.setQueryData(["cabins"], ctx.previous);
}
```

---

## 🔁 Other Important Methods

### ♻️ `refetchQueries`

```ts
queryClient.refetchQueries({ queryKey: ["cabins"] });
```

Forces refetch immediately.

---

### 🧹 `removeQueries`

```ts
queryClient.removeQueries({ queryKey: ["cabins"] });
```

Deletes queries from cache entirely.

---

### ⏹️ `cancelQueries`

```ts
queryClient.cancelQueries({ queryKey: ["cabins"] });
```

Stops in-flight requests.

---

## 🧠 Stability and Lifecycle

- The Query Client is **stable**
    
- Same instance for the app lifetime
    
- Safe to call methods anytime
    
- Does not trigger renders by itself
    

Renders happen because **queries subscribed to cache change**.

---

## ⚠️ Common Mistakes

### 🚫 Updating Cache with Wrong Query Key

If keys don’t match exactly → UI won’t update.

---

### 🚫 Overusing `invalidateQueries`

Sometimes `setQueryData` is faster and cleaner.

---

### 🚫 Treating It Like Global State

It manages **server state**, not UI state.

---

## 🧠 Mental Model

> `useQuery` subscribes to data  
> `useMutation` changes data  
> `useQueryClient` coordinates everything

---

## 🧠 Comparison with Redux

| React Query       | Redux              |
| ----------------- | ------------------ |
| Query Client      | Store              |
| Query Key         | Slice key          |
| invalidateQueries | Dispatch refetch   |
| setQueryData      | Optimistic reducer |

---

## 🧠 One-Sentence Summary

> `useQueryClient` provides imperative access to the central React Query cache, allowing you to invalidate, update, read, and coordinate queries in response to mutations or user actions.