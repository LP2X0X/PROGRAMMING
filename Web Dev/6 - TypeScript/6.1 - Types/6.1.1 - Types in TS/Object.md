---
tags: 
 - typescript
 - type
 - object
---

### 🔹 Definition

In TypeScript, **`object`** refers to _any non-primitive value_.  
That means any value that’s **not** a number, string, boolean, bigint, symbol, null, or undefined.

```ts
let obj: object;
obj = { name: "Alice" }; // ✅
obj = [1, 2, 3];         // ✅
obj = () => {};          // ✅
obj = null;              // ❌ (null is not an object in TS)
obj = 42;                // ❌ (primitive)
```

---

### 🔹 Key characteristics

|Feature|Explanation|
|---|---|
|`object` is broad|It only ensures the value is _not primitive_|
|Can’t access properties|TS doesn’t know what’s inside|
|Used for generic “object-like” types|e.g. `{}`, arrays, class instances, functions|

---

### 🪤 Pitfall

You **can’t** access properties directly when using `object`:

```ts
let person: object = { name: "Bob" };
console.log(person.name); // ❌ Property 'name' does not exist
```

✅ Fix by using a **type annotation** or **interface**:

```ts
let person: { name: string } = { name: "Bob" };
console.log(person.name); // ✅
```

---

### 💡 Tips

- Use `object` **only** when you don’t care about internal structure.
    
- For structured objects, always use `{}` type, interface, or a type alias.
    
- Never confuse with `{}` — `{}` means _any non-nullish value_, including primitives.
    

|Type|Allows|Example|
|---|---|---|
|`object`|Any non-primitive|`{}`, `[]`, `() => {}`|
|`{}`|Anything but `null`/`undefined`|`42`, `"hi"`, `{}`|

---

### 🧭 Example

```ts
function log(obj: object) {
  console.log(obj);
}

log({ key: "value" }); // ✅
log([1, 2, 3]);        // ✅
log(10);               // ❌
```

