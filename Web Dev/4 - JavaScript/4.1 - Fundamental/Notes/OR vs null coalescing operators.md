---
tags:
  - js
  - note
  - fundamental
---

- The important difference between [[Web Dev/4 - JavaScript/4.1 - Fundamental/4.1.3 - Operators/Logical Operators/OR| OR operator]] and [[nullish coalescing operator]]  is that:
	- `||` returns the first _truthy_ value.
	- `??` returns the first _defined_ value.
- In other words, `||` doesn’t distinguish between `false`, `0`, an empty string `""` and `null/undefined`. They are all the same – falsy values. If any of these is the first argument of `||`, then we’ll get the second argument as the result.

---

## Example 1: Boolean `false`

```ts
const value = false;

const resultOr = value || "fallback";
const resultNullish = value ?? "fallback";

resultOr;      // "fallback"
resultNullish; // false
```

Explanation:  
`false` is **falsy**, so `||` skips it.  
`false` is **defined**, so `??` keeps it.

---

## Example 2: Number `0`

```ts
const count = 0;

count || 10;   // 10
count ?? 10;   // 0
```

Explanation:  
`0` is falsy → `||` falls back.  
`0` is not `null` or `undefined` → `??` keeps it.

---

## Example 3: Empty string `""`

```ts
const name = "";

name || "Anonymous"; // "Anonymous"
name ?? "Anonymous"; // ""
```

Explanation:  
Empty string is falsy, but still a valid value.

---

## Example 4: `null`

```ts
const data = null;

data || "default"; // "default"
data ?? "default"; // "default"
```

Explanation:  
`null` is both falsy **and** nullish, so both operators fall back.

---

## Example 5: `undefined`

```ts
let value;

value || "fallback"; // "fallback"
value ?? "fallback"; // "fallback"
```

Explanation:  
Same behavior as `null`.

---

## Summary Table

| Value       | `value || "x"` | `value ?? "x"` |
| ----------- | -------------- | -------------- |
| `false`     | `"x"`          | `false`        |
| `0`         | `"x"`          | `0`            |
| `""`        | `"x"`          | `""`           |
| `null`      | `"x"`          | `"x"`          |
| `undefined` | `"x"`          | `"x"`          |

---

## Rule of Thumb

- Use **`||`** when _any falsy value_ should trigger a fallback.
    
- Use **`??`** when _only `null` or `undefined`_ should trigger a fallback.
    

This distinction is especially important when `0`, `false`, or `""` are **valid and meaningful values**.