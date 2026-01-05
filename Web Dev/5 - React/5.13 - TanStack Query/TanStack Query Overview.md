---
tags: 
 - react
 - query
 - overview
---

## 🧠 What Is React Query?

**React Query** (now **TanStack Query**) is a library for managing **server state** in React.

> Server state = data that lives on a remote server and can change independently of your UI.

Examples:

- API responses
    
- Paginated lists
    
- Search results
    
- User profiles from backend
    

![[Pasted image 20260104151557.png|center|700]]

---

## 🧩 The Core Problem React Query Solves

Without React Query, you must manually handle:

- Loading state
    
- Error state
    
- Caching
    
- Deduplication
    
- Refetching
    
- Race conditions
    
- Stale data
    

React Query **centralizes and standardizes** these concerns.

---

## 🗂️ Client vs Server State (Critical Distinction)

| State Type    | Example           | Where It Belongs    |
| ------------- | ----------------- | ------------------- |
| UI state      | Modal open, theme | React state / Redux |
| Server state  | Users list        | React Query         |
| Derived state | Filtered list     | Selectors           |
| URL state     | Filters, IDs      | React Router        |

React Query is **only** for server state.

---

## 🚀 Basic Setup

```ts
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

root.render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
);
```

This creates a **global cache**.

---

## 🔑 Queries: Reading Data

```ts
const { data, isLoading, error } = useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});
```

### What happens internally

1. Check cache
    
2. If fresh → return immediately
    
3. If stale → refetch
    
4. Deduplicate parallel requests
    

---

## 🧾 Query Keys (Identity of Data)

```ts
["users", userId]
```

Rules:

- Must be **stable**
    
- Defines cache identity
    
- Different keys = different cache entries
    

Think of keys as **primary keys for API data**.

---

## 🧠 Stale vs Fresh Data

React Query separates:

- **Freshness** (`staleTime`)
    
- **Cache lifetime** (`cacheTime` / `gcTime`)
    

```ts
staleTime: 5 * 60 * 1000
```

- Fresh data → no refetch
    
- Stale data → background refetch
    

This enables **instant UI + eventual consistency**.

---

## 🔄 Background Refetching

Triggers refetch when:

- Window refocus
    
- Network reconnect
    
- Component remount
    
- Interval polling
    

This is **automatic**, no code needed.

---

## 🧪 Mutations: Writing Data

```ts
const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries(["users"]);
  },
});
```

Mutations:

- Do not cache results
    
- Trigger side effects
    
- Invalidate affected queries
    

---

## ⚡ Optimistic Updates

```ts
onMutate: async (newUser) => {
  await queryClient.cancelQueries(["users"]);
  const prev = queryClient.getQueryData(["users"]);
  queryClient.setQueryData(["users"], [...prev, newUser]);
  return { prev };
}
```

Used for:

- Fast UI feedback
    
- Rollback on failure
    

---

## 🔗 Dependent & Conditional Queries

```ts
useQuery({
  queryKey: ["profile", userId],
  queryFn: fetchProfile,
  enabled: !!userId,
});
```

- Query runs **only when condition is true**
    
- Prevents invalid requests
    

---

## 🧭 Pagination & Infinite Queries

```ts
useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam }) => fetchPosts(pageParam),
});
```

Handles:

- Cursor-based APIs
    
- Infinite scroll
    
- Page accumulation
    

---

## 🧱 Error Handling

```ts
if (error instanceof Error) {
  error.message;
}
```

Supports:

- Retry logic
    
- Exponential backoff
    
- Global error boundaries
    

---

## 🧩 Suspense Integration

```ts
useQuery({
  suspense: true,
});
```

- Enables render-as-you-fetch
    
- Must use `<Suspense>`
    
- Router-agnostic
    

⚠️ Requires careful architecture.

---

## 🧠 React Query vs Redux Toolkit

| Feature             | React Query | Redux |
| ------------------- | ----------- | ----- |
| Server state        | ✅          | ❌    |
| Caching             | ✅          | ❌    |
| UI state            | ❌          | ✅    |
| Boilerplate         | Low         | High  |
| Async orchestration | ❌          | ✅    |

They solve **different problems**.

---

## 🧠 React Query vs Router Loaders

|Feature|React Query|Router Loader|
|---|---|---|
|Fetch before render|❌*|✅|
|Cache sharing|✅|⚠️|
|Background refetch|✅|❌|
|Mutations|✅|⚠️|
|Navigation awareness|❌|✅|

* unless using Suspense carefully.

---

## 🧠 Best Practices

- One query = one resource
    
- Keep query functions pure
    
- Use invalidation, not manual refetch
    
- Avoid storing server data in Redux
    
- Prefer query keys over global state
    

---

## 🧠 When NOT to Use React Query

- Pure UI state
    
- Local-only data
    
- Simple apps with no async complexity
    
- When router loaders fully cover your needs
    

---

## 🧠 Key Takeaways

- React Query manages **server state**
    
- It removes async boilerplate
    
- It provides caching, refetching, and consistency
    
- It complements React Router and Redux
    
- It is the **default choice** for modern React apps
    
