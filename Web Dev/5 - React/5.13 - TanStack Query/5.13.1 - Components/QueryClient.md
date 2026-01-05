---
tags: 
 - react
 - query
 - object
---

Below is a **detailed, note-style explanation of the React Query `QueryClient` (often informally called “create client”)**, with **emoji in section headers** and a **clear mental model**.

---

## 🧠 What Is `QueryClient`?

`QueryClient` is the **central manager** of all React Query behavior.

It is responsible for:

- Query cache
    
- Mutation cache
    
- Fetch lifecycle
    
- Refetch rules
    
- Garbage collection
    
- Global defaults
    

Think of it as **the brain of React Query**.

---

## 🏗️ Creating a Query Client

```ts
import { QueryClient } from "@tanstack/react-query";

const queryClient = new QueryClient();
```

This creates an **isolated cache instance**.

> One app = one `QueryClient` (almost always)

---

## 🔌 Providing the Client

```tsx
import { QueryClientProvider } from "@tanstack/react-query";

<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

This makes the client available to:

- `useQuery`
    
- `useMutation`
    
- `useInfiniteQuery`
    

Without this provider, hooks will throw errors.

---

## ⚙️ Default Options (Global Configuration)

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
      retry: 2,
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

### Why this matters

- Avoid repeating options in every hook
    
- Enforce consistent data behavior
    
- Tune performance globally
    

---

## 🧪 Query Cache Management

`QueryClient` owns a **query cache**.

```ts
queryClient.getQueryData(["users"]);
queryClient.setQueryData(["users"], newData);
queryClient.invalidateQueries(["users"]);
```

Use cases:

- Manual cache updates
    
- Optimistic UI
    
- Force refetch
    

---

## 🔄 Invalidation vs Refetch

```ts
queryClient.invalidateQueries(["users"]);
```

- Marks data as **stale**
    
- Triggers refetch on next render or focus
    

Prefer invalidation over:

```ts
queryClient.refetchQueries()
```

---

## 🧹 Garbage Collection (Cache Lifetime)

React Query removes unused data automatically.

```ts
cacheTime: 10 * 60 * 1000
```

- After last observer unmounts
    
- Prevents memory leaks
    
- Does NOT affect freshness
    

---

## 🔁 Multiple Clients (Advanced Use)

```ts
const adminClient = new QueryClient();
const publicClient = new QueryClient();
```

Used for:

- Multi-tenant apps
    
- Micro-frontends
    
- Auth-separated data
    

⚠️ Advanced scenario only.

---

## 🧠 Why Hooks Don’t “Create” Clients

Hooks like `useQuery` **do not create state**.

- Client must exist **outside React**
    
- Must persist across renders
    
- Must not be recreated per render
    

Creating it inside a component is a bug.

---

## 🚫 Common Mistakes

❌ Creating client inside component  
❌ Multiple clients unintentionally  
❌ No global defaults  
❌ Manually storing server data in Redux

---

## 🧠 QueryClient vs Redux Store

| Aspect             | QueryClient  | Redux Store    |
| ------------------ | ------------ | -------------- |
| Purpose            | Server state | UI / app state |
| Immutable reducers | ❌           | ✅             |
| Caching            | ✅           | ❌             |
| Async handling     | Built-in     | Manual         |
| Persistence        | Cache-based  | Manual         |

They are complementary.

---

## 🧠 Mental Model

- `QueryClient` = cache + scheduler
    
- Queries subscribe to cache entries
    
- Mutations invalidate queries
    
- UI reacts to cache changes
    

No reducers. No actions. No boilerplate.

---

## 🧠 Key Takeaways

- `QueryClient` is mandatory
    
- It owns all React Query state
    
- Configure it once at app startup
    
- Use defaults to avoid repetition
    
- Never recreate it during render
    