---
tags: 
 - typescript
 - note
---

### The Problem

When you initialize a variable with an object, TypeScript infers the types of the properties to be as general as possible so that you can change them later.

For example, if you write:

```ts
const req = { url: "https://example.com", method: "GET" };
```

TypeScript infers `req` to be:

```ts
{
  url: string;
  method: string;
}
```

Even though you wrote `"GET"`, TypeScript assumes `req.method` is just a `string` because you might want to change it to `"POST"` or `"whatever"` later.

### The Conflict

This becomes a problem if you try to pass that property to a function that expects a **specific literal type**.

Imagine you have a function like this:

```ts
function handleRequest(url: string, method: "GET" | "POST") {
  // ...
}

const req = { url: "https://example.com", method: "GET" };

// ❌ ERROR
handleRequest(req.url, req.method);
```

Why creates an error?

TypeScript complains: Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.

Because req.method is inferred as string, it could theoretically be "GUESS" or "DELETE", which aren't allowed by the function.

### The Solutions

The Handbook offers two main ways to tell TypeScript, "I promise this string really is 'GET'."

#### 1. Type Assertion

You can force the type to be a literal when you create the object or when you use it.

**Option A (At creation):**

```ts
// "I specifically want method to be the literal type 'GET'"
const req = { url: "https://example.com", method: "GET" as "GET" };
```

**Option B (At call site):**

```ts
// "I know req.method is actually 'GET' right now"
handleRequest(req.url, req.method as "GET");
```

#### 2. `as const` (The Best Practice)

This is the most powerful solution. You can convert the entire object to be purely literals by using `as const`.

```ts
const req = { url: "https://example.com", method: "GET" } as const;
```

When you use `as const`:

1. All properties become `readonly`.
    
2. All primitives are inferred as their **literal types** (e.g., `method` is `"GET"`, not `string`).
    

Now `req` is inferred as:

```ts
{
  readonly url: "https://example.com";
  readonly method: "GET";
}
```

This works perfectly with `handleRequest` because `"GET"` is definitely assignable to `"GET" | "POST"`.