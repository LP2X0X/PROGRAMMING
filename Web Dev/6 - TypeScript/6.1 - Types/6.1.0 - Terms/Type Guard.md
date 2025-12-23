---
tags: 
 - ts
 - term
---

A **type guard** is a runtime check that tells TypeScript **which specific type a value has**, allowing safe access to properties or methods of that type.

```ad-note
The special checks are called _type guards_ and the process of refining types to more specific types than declared is called _narrowing_.
```


### Example

```ts
function printValue(value: string | number) {
  if (typeof value === "string") {
    // value is narrowed to string
    console.log(value.toUpperCase());
  } else {
    // value is narrowed to number
    console.log(value.toFixed(2));
  }
}
```

Here, `typeof value === "string"` is a type guard.  
It narrows `value` from `string | number` to a specific type within each branch.

### Custom Type Guard

```ts
function isUser(obj: any): obj is User {
  return obj && typeof obj.name === "string";
}
```

This lets TypeScript understand the type after the check.

**In short:** type guards connect runtime checks with compile-time type safety.