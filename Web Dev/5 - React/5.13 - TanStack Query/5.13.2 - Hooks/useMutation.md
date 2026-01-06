---
tags: 
 - web dev
---

## 🧠 What `useMutation` Is (Conceptual)

`useMutation` is a hook for **imperative, side-effectful operations** that **change server state**.

Typical examples:

- Create
    
- Update
    
- Delete
    
- Submit forms
    
- Trigger actions
    

Unlike `useQuery`, it **does not run automatically**.

---

## 🔁 Mutation vs Query (Core Distinction)

| Aspect    | `useQuery` | `useMutation`   |
| --------- | ---------- | --------------- |
| Execution | Automatic  | Manual          |
| Trigger   | Render     | `mutate()`      |
| Purpose   | Read data  | Change data     |
| Caching   | Yes        | No (by default) |
| Refetch   | Built-in   | Manual          |

---

## ⚙️ Basic API Shape

```ts
const mutation = useMutation({
  mutationFn,
  onSuccess,
  onError,
  onSettled,
});
```

Returned object (simplified):

```ts
{
  mutate,
  mutateAsync,
  isLoading,
  isSuccess,
  isError,
  error,
  data,
  reset
}
```

---

## 🧩 Internal State Machine

`useMutation` tracks **one mutation instance** with a lifecycle:

```
idle → loading → success | error
```

State fields:

- `status`
    
- `data`
    
- `error`
    

Derived booleans:

- `isIdle`
    
- `isLoading`
    
- `isSuccess`
    
- `isError`
    

---

## 🧠 How `mutate()` Works (Internals)

Conceptually:

```ts
function mutate(variables) {
  setStatus("loading");

  Promise.resolve(mutationFn(variables))
    .then((data) => {
      setStatus("success");
      setData(data);
    })
    .catch((error) => {
      setStatus("error");
      setError(error);
    });
}
```

Key points:

- `mutate` closes over internal state
    
- State update triggers re-render
    
- Status flags recompute automatically
    

---

## 🔄 Why Mutations Are Imperative

```ts
<button onClick={() => mutate(id)}>Delete</button>
```

You explicitly decide:

- **When** to run
    
- **With what data**
    

This contrasts with `useQuery`, which is **declarative**.

---

## 📦 `mutationFn`

### 📍 Definition

The function that performs the async side effect.

```ts
const deleteCabin = (id: number) =>
  fetch(`/cabins/${id}`, { method: "DELETE" });
```

### Rules:

- Must return a Promise
    
- Receives variables passed to `mutate`
    

---

## 🎯 `mutate` vs `mutateAsync`

|Method|Behavior|
|---|---|
|`mutate`|Fire-and-forget|
|`mutateAsync`|Returns a Promise|

```ts
await mutateAsync(id);
```

Use `mutateAsync` when you need:

- Sequential logic
    
- Try/catch
    
- Conditional flows
    

---

## 🧭 Lifecycle Callbacks

### ✅ `onSuccess`

```ts
onSuccess: () => {
  queryClient.invalidateQueries(["cabins"]);
}
```

Used for:

- Cache invalidation
    
- Navigation
    
- Notifications
    

---

### ❌ `onError`

```ts
onError: (error) => {
  toast.error(error.message);
}
```

---

### 🔚 `onSettled`

Runs **regardless of outcome**.

```ts
onSettled: () => {
  closeModal();
}
```

---

## 🔄 Cache Interaction

Mutations **do not update cache automatically**.

You must explicitly:

- `invalidateQueries`
    
- `setQueryData`
    
- `refetchQueries`
    

```ts
queryClient.invalidateQueries({ queryKey: ["cabins"] });
```

---

## 🧠 Optimistic Updates

`useMutation` supports optimistic UI via `onMutate`.

```ts
onMutate: async (id) => {
  await queryClient.cancelQueries(["cabins"]);

  const prev = queryClient.getQueryData(["cabins"]);

  queryClient.setQueryData(["cabins"], (old) =>
    old.filter((c) => c.id !== id)
  );

  return { prev };
}
```

Rollback on error:

```ts
onError: (_err, _vars, ctx) => {
  queryClient.setQueryData(["cabins"], ctx.prev);
}
```

---

## ⚠️ Common Pitfalls

### 🚫 Using `useMutation` for Fetching Data

Wrong:

```ts
useMutation(fetchUsers);
```

Correct:

```ts
useQuery(["users"], fetchUsers);
```

---

### 🚫 Forgetting Cache Updates

Mutation succeeds but UI stays stale.

Fix:

```ts
invalidateQueries
```

---

### 🚫 Overusing Mutations

Do not replace all server logic with mutations.  
Use them **only for writes**.

---

## 🧠 Stability Guarantees

| Value        | Stability             |
| ------------ | --------------------- |
| `mutate`     | Stable across renders |
| Status flags | Change per render     |
| `mutationFn` | Re-read per hook call | 

---

## 🧠 Mental Model

> `useMutation` is a **command**, not a subscription.

You issue it, React Query tracks it, and you respond to its outcome.

---

## 🧠 One-Sentence Summary

> `useMutation` manages the lifecycle of imperative async operations, exposing state flags derived from an internal state machine that updates when `mutate()` is called.