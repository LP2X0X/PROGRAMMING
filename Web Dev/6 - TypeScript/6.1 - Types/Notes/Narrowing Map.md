---
tags: 
 - typescript
 - narrowing
 - note
---

````ad-question
Why `has` method does not help narrowing the item with "error" key here:
```ts
type Event = {
  message: string;
};

const processUserMap = (eventMap: Map<string, Event>) => {
  if (eventMap.has("error")) {
    const message = eventMap.get("error").message;

    throw new Error(message);
  }
};
```
````

````ad-Answer
```ts
type Event = {
  message: string;
};

const processUserMap = (eventMap: Map<string, Event>) => {
  const event = eventMap.get("error");
  if (event) {
    const message = event.message;

    throw new Error(message);
  }
};
```
````

This is a classic "TypeScript doesn't know what you know" situation.

The reason it fails is that TypeScript treats `has()` and `get()` as **two completely independent method calls**.

To the compiler, there is no link between the two lines of code. It does not understand that `has("error")` returning `true` guarantees that `get("error")` will return a value.

### The Compiler's Perspective

1. **Line 1 (`has`):** You call a method that returns a `boolean`. The compiler checks the if-statement and says, "Okay, we proceed only if this is true."
    
2. **Line 2 (`get`):** You call a _different_ method. The type definition for `.get()` in the TypeScript library looks like this:
    
    ```ts
    get(key: K): V | undefined;
    ```
    
    It is hard-coded to **always** return `V | undefined` because looking up a key in a Map is inherently risky.
    

The compiler does not track the "state" of the Map or correlate the parameters between the two distinct method calls. It plays it safe and assumes `get()` could still return `undefined`.

### The Correct Pattern

Instead of "ask then grab," you should "grab then check."

You must assign the result of `.get()` to a variable first, then check if that variable exists. TypeScript can narrow the type of the **variable** easily.

```ts
const processUserMap = (eventMap: Map<string, Event>) => {
  const errorEvent = eventMap.get("error"); // Type: Event | undefined

  // TypeScript narrows 'errorEvent' because it is a direct variable check
  if (errorEvent) {
    throw new Error(errorEvent.message); // Safe! Type is 'Event'
  }
};
```

### Summary

- **`.has()`** is useful for logic logic (e.g. "Do we have a record?"), but it does not narrow types.
    
- **`.get()`** always returns a potentially undefined value.
    
- **The Fix:** Always store the `.get()` result in a `const` and check that const.
    