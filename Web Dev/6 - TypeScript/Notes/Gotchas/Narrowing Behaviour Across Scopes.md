---
tags: 
 - typescript
 - narrowing
 - note
---

````ad-question
Why the searchParams.name property still has type of string | undefined inside the if statement scope?
```ts
const findUsersByName = (
  searchParams: { name?: string },
  users: {
    id: string;
    name: string;
  }[]
) => {
  if (searchParams.name) {
    return users.filter((user) => user.name.includes(searchParams.name));
  }

  return users;
};
```
````

This is a classic TypeScript "gotcha." The reason narrowing fails here is because of **function boundaries (closures)** and the potential for **mutability**.

TypeScript is being "paranoid" for your safety. Here is exactly what is happening:

### The Problem: "The Paranoid Compiler"

When you write `if (searchParams.name)`, TypeScript correctly knows that **inside that `if` block**, `searchParams.name` is a string.

However, when you enter the arrow function inside `filter(...)`, you are entering a **new function scope**.

```ts
return users.filter((user) => {
  // TypeScript Look: "I am inside a callback function now."
  // "I cannot guarantee that 'searchParams.name' hasn't changed
  // between the time you checked it and the time this function runs."
  return user.name.includes(searchParams.name); // Error!
});
```

Even though _we_ know `Array.filter` runs immediately (synchronously), TypeScript treats callbacks conservatively. It worries that `searchParams` is an object, and objects are mutable. Theoretically, some other code _could_ set `searchParams.name = undefined` right before the callback executes.

Because TypeScript cannot 100% guarantee the property on the object stays narrowed inside the callback, it resets the type back to `string | undefined`.

### The Fix: Alias to a `const`

The standard way to fix this is to save the narrowed value into a `const` variable before entering the function. Since a `const` cannot be reassigned, TypeScript trusts it completely.

```ts
const findUsersByName = (
  searchParams: { name?: string },
  users: { id: string; name: string }[]
) => {
  if (searchParams.name) {
    // 1. Save the narrowed value to a const
    // TS knows 'query' is definitely a string and cannot change.
    const query = searchParams.name;

    return users.filter((user) => user.name.includes(query)); // ✅ Works perfectly
  }

  return users;
};
```

### Alternative Fix: [[Non-null Assertion|Non-Null Assertion (!)]]

If you want to be quick and you are absolutely certain the value is there (which you are, because of the `if`), you can tell TypeScript "trust me" using the `!` operator:

```ts
// valid, but arguably less clean than the const approach
return users.filter((user) => user.name.includes(searchParams.name!));
```

### Summary

- **Narrowing checks** (like `if`) work on the **current scope**.
    
- **Callbacks** create a **new scope**.
    
- TypeScript drops the narrowing inside the callback because it fears the object property might have changed.
    
- **Solution:** Assign the checked value to a `const` variable, which carries the narrowed type safely into the callback.
    


----

- **Nested Scopes (Regular blocks like `if`, `for`):** TypeScript trusts these completely. It knows the code runs immediately and sequentially, so it **keeps** the narrowing.
    
- **Function Scopes (Callbacks):** TypeScript does **not** trust these. It considers them a "boundary" where side effects could happen, so it **resets** the narrowing for object properties.
    

Here is the direct comparison to prove it:

### 1. Nested `if` (Safe ✅)

TypeScript looks at this and sees one straight line of execution. It knows nothing can change `searchParams.name` between the first `{` and the second `{`.

```ts
const testNestedIf = (searchParams: { name?: string }) => {
  if (searchParams.name) {
    // TS: "Okay, name is a string here."
    
    if (true) {
       // TS: "I'm still inside the same synchronous flow."
       // "name is STILL a string."
       console.log(searchParams.name.toUpperCase()); // ✅ Works!
    }
  }
};
```

### 2. Callback Function (Unsafe ❌)

TypeScript looks at this and sees a "black box." It worries that `searchParams` (which is an object reference) might be modified by some other part of your code while the array loops.

```ts
const testCallback = (
  searchParams: { name?: string }, 
  users: any[]
) => {
  if (searchParams.name) {
    // TS: "Okay, name is a string here."

    users.forEach(() => {
       // TS: "Whoa, we crossed a function boundary!"
       // "Something outside might have set searchParams.name = undefined."
       // "I'm resetting it to (string | undefined) just to be safe."
       console.log(searchParams.name.toUpperCase()); // ❌ Error!
    });
  }
};
```

### Why the difference?

It comes down to **Mutability** (the ability to change) + **Timing**.

1. **Objects are Mutable:** In JavaScript, if you pass an object around, anyone can change its properties.
    
2. **Callbacks are unpredictable:** Even though _we_ know `.filter` runs immediately, strictly speaking, functions are designed to be run "whenever."
    

TypeScript follows this rule: **"If I step into a new function, I stop trusting that object properties haven't changed."**

### Summary

| **Scope Type**                    | **Does Narrowing Persist?** | **Why?**                                                         |
| --------------------------------- | --------------------------- | ---------------------------------------------------------------- |
| **Nested `if` / `else`**          | **Yes** ✅                  | Execution is linear and guaranteed.                              |
| **`for` / `while` loops**         | **Yes** ✅                  | Execution is linear and guaranteed.                              |
| **Callbacks (`.map`, `.filter`)** | **No** ❌                   | Function boundaries break the trust chain for object properties. |

---

If TypeScript was 100% "paranoid" everywhere, you would have to re-verify every variable on every single line of code, which would make the language unusable. To solve this, TypeScript uses a concept called **Control Flow Analysis**.

Here is why it trusts a nested `if` but distrusts a callback:

----

### 1. The "Synchronous Execution" Assumption

Within a single function body, TypeScript assumes a **linear timeline**. It assumes that between line 1 and line 2, your object property won't change unless _this specific piece of code_ changes it.

```ts
if (searchParams.name) {
  // Line A: verified as string
  console.log(searchParams.name.length); 
  // Line B: TS assumes no other code ran between Line A and B.
}
```

Technically, if you had a `setInterval` running in the background changing `searchParams.name` every millisecond, Line B **could** fail. But TypeScript ignores this possibility because it’s a "race condition"—it assumes your code is running in a predictable, synchronous sequence.

---

### 2. The "Function Boundary" Reset

The moment you define a function (like the callback in `.filter`), you are creating a block of code that **might be executed later**.

Even though `.filter` is synchronous, TypeScript's general rule for functions is: **"I don't know when this code will actually run."**

```ts
if (searchParams.name) {
  const callback = () => console.log(searchParams.name);
  
  // What if I do this?
  searchParams.name = undefined; 
  
  // And then call it?
  callback(); // This would crash if TS didn't warn you!
}
```

Because TypeScript sees the callback as a separate "unit" of code, it resets the narrowing for all **mutable references** (objects) to be safe.

---

### Why does it trust `const` but not `searchParams.name`?

This is the key to understanding the difference:

- **`searchParams` is an object:** Even if `searchParams` is a `const`, its **properties** (like `.name`) are mutable. Anyone with a reference to that object can change the value.
    
- **A local `const name = searchParams.name`:** This is a **primitive value binding**. Once you've copied the string into a local `const`, it is physically impossible for that specific variable to change to `undefined`.
    

### Summary of Trust

| **Scenario**        | **Does TS trust it?** | **Why?**                                                         |
| ------------------- | --------------------- | ---------------------------------------------------------------- |
| **Local `const`**   | **Total Trust**       | It literally cannot be reassigned.                               |
| **Local `let`**     | **High Trust**        | TS tracks if _you_ changed it in the current block.              |
| **Object Property** | **Conditional Trust** | Trusted in the current block, but reset inside any new function. |
| **Global Variable** | **Low Trust**         | Same as object properties; very volatile.                        |

----

### Reference
https://stackoverflow.com/questions/71420195/why-sometimes-need-to-do-type-narrowing-again-inside-a-child-function-scope