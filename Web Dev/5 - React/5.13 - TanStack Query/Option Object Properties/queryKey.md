---
tags: 
 - react
 - query
 - option
 - object
 - property
---

In TanStack Query (formerly React Query), the **`queryKey`** is the most fundamental concept. It acts as a unique identifier for your data, functioning similarly to a **key in a cache** or a **dependency array in `useEffect`**.

---

## 1. What is the `queryKey`?

At its core, the `queryKey` tells React Query how to label the data it fetches. If two components use the same `queryKey`, they will share the same data, loading state, and cached results.

### Syntax

In v4 and v5, the `queryKey` **must be an array**, even if it only contains one string.

```js
// Simple key
useQuery({ queryKey: ['todos'], ... })

// Complex key (with variables)
useQuery({ queryKey: ['todo', 5], ... })

// Key with object
useQuery({ queryKey: ['todo', { id: 5, status: 'done' }], ... })
```

---

## 2. The "Dependency Array" Behavior

The `queryKey` isn't just a label; it’s a trigger. When any part of the `queryKey` changes, React Query automatically assumes the data is stale and triggers a **refetch**.

### Example:

```js
function TodoDetail({ todoId }) {
  return useQuery({
    queryKey: ['todo', todoId], // When todoId changes, React Query fetches new data
    queryFn: () => fetchTodoById(todoId),
  });
}
```

---

## 3. Best Practices 

### A. Treat it like a Hierarchy

Organize your keys from **most general** to **most specific**. This allows you to perform "partial matching" when invalidating data.

- `['todos']` — All todos.
    
- `['todos', 'list']` — Just the list view.
    
- `['todos', 'detail', 1]` — A specific todo.
    

### B. Use Query Key Factories

As your app grows, hardcoding strings like `'todos'` everywhere leads to typos and bugs. Use a factory object to manage them in one place.

```js
const todoKeys = {
  all: ['todos'],
  lists: () => [...todoKeys.all, 'list'],
  list: (filter) => [...todoKeys.lists(), { filter }],
  details: () => [...todoKeys.all, 'detail'],
  detail: (id) => [...todoKeys.details(), id],
};

// Usage
useQuery({ queryKey: todoKeys.detail(props.id), ... });
```

### C. Objects vs. Strings

React Query uses **deterministic hashing** for objects in keys. This means the order of properties in an object doesn't matter, but the order of items in the array _does_.

- `['todos', { status, page }]` is the same as `['todos', { page, status }]`.
    
- `['todos', 'active']` is **NOT** the same as `['active', 'todos']`.
    

---

## 4. Why is the Key so Important?

### Manual Invalidation

When you update data (via `useMutation`), you use the `queryKey` to tell React Query which cache entries are now "trash" and need to be reloaded.

```js
const queryClient = useQueryClient();

// After adding a new todo, refresh everything under the 'todos' key
queryClient.invalidateQueries({ queryKey: ['todos'] });
```

---

## 5. Summary Checklist

- [ ] **Always an Array:** Even for single strings: `['todos']`.
    
- [ ] **Include Dependencies:** If your `queryFn` uses a variable, that variable **must** be in the `queryKey`.
    
- [ ] **Unique Labels:** Ensure different types of data (e.g., `users` vs `posts`) have different starting strings.
    
- [ ] **Serializability:** Keys should be JSON-serializable (avoid passing functions or complex class instances).
    