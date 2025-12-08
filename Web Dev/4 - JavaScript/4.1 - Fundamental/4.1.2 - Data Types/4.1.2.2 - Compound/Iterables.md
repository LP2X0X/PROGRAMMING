---
tags:
  - js
  - term
  - fundamental
---

In JavaScript, an **iterable** is any object that implements the `@@iterator` **method** (i.e., has a method with the key `[Symbol.iterator]`), which returns an **iterator**.

At a high level, the **Iterable** concept separates data structures (data sources) from the logic used to traverse them (data consumers).

- **Iterable (The Data Source):** An object that _can_ be iterated over (e.g., an Array, a String). It "knows" how to produce an Iterator.
    
- **Iterator (The Pointer):** An object that tracks the current position in the sequence and knows how to access the next item.
    
- **Consumer:** Syntax that automatically handles the iteration (e.g., `for...of`, `...` spread).
    

Here’s how to implement a **custom iterable**:

--- 

### ✅ **Minimal Requirements to Be Iterable**

```js
const obj = {
  [Symbol.iterator]() {
    return {
      next() {
        return { value: ..., done: ... };
      }
    };
  }
};

```

| Syntax Part                      | Must-Have? | Purpose                                                           |
| -------------------------------- | ---------- | ----------------------------------------------------------------- |
| `[Symbol.iterator]()`            | ✅ Yes     | This is what defines the object as **iterable**.                  |
| `next()`                         | ✅ Yes     | Required on the iterator object to control the iteration process. |
| `{ value: ..., done: ... }`      | ✅ Yes     | The `next()` method must return this object to indicate state.    |
| `value` (inside returned object) | ✅ Yes     | The actual value for the current iteration step.                  |
| `done` (inside returned object)  | ✅ Yes     | A boolean telling whether iteration is complete.                  |

---

## The Protocols

JavaScript defines iteration using two specific protocols (interfaces). An object is only "iterable" if it implements the first protocol.

### A. The Iterable Protocol

To be **iterable**, an object must have a method stored under the special symbol key `[Symbol.iterator]`.

- **Input:** None.
    
- **Output:** Must return an **Iterator** object.
    

```js
const myIterable = {
  [Symbol.iterator]() {
    // return an Iterator object here
  }
};
```

### B. The Iterator Protocol

To be an **iterator**, an object must implement a `.next()` method.

- **Input:** None (usually).
    
- **Output:** An object (IteratorResult) with two properties:
    
    - `value`: The data for the current step (can be any type).
        
    - `done`: A boolean (`false` if reading, `true` if finished).
        

```js
// A simple iterator object
const myIterator = {
  next() {
    return { value: 1, done: false }; // Yields 1
    // eventually: return { value: undefined, done: true };
  }
};
```

---

### ✅ Common Iterable Types:


JavaScript has many native objects that implement `Symbol.iterator` by default:

- **Arrays & TypedArrays:** `[1, 2, 3]`
    
- **[[Web Dev/4 - JavaScript/4.1 - Fundamental/4.1.2 - Data Types/4.1.2.1 - Primitives/Strings/String|Strings]]:** `"hello"` (Iterates over Unicode code points)
    
- **[[Map|Maps]]:** Iterates over `[key, value]` entries.
    
- **[[Set|Sets]]:** Iterates over values.
    
- **Arguments object:** Inside functions.
    
- **DOM Collections:** `NodeList` (e.g., from `document.querySelectorAll`).
    

> **Note:** Plain Objects (`{}`) are **NOT** iterable by default. You cannot use `for...of` on them directly without adding a custom `Symbol.iterator`.

---

## Consuming Iterables

The following syntax constructs automatically look for `Symbol.iterator` and call `.next()` until `done: true`.

|**Consumer**|**Example**|**Description**|
|---|---|---|
|**`for...of`**|`for (const x of iterable) {}`|The standard loop for data.|
|**Spread Syntax**|`[...iterable]`|Expands iterable into an Array.|
|**Destructuring**|`const [a, b] = iterable`|Assigns first N values to variables.|
|**`Array.from()`**|`Array.from(iterable)`|Converts any iterable to a real Array.|
|**`yield*`**|`yield* iterable`|Delegates generation to another iterable.|
|**Constructors**|`new Set(iterable)`|Maps/Sets accept iterables in constructor.|


---

### ✅ Basic Custom Iterable Example

```js
let range = {
  from: 1,
  to: 5
};

// 1. call to for..of initially calls this
range[Symbol.iterator] = function() {

  // ...it returns the iterator object:
  // 2. Onward, for..of works only with the iterator object below, asking it for next values
  return {
    current: this.from,
    last: this.to,

    // 3. next() is called on each iteration by the for..of loop
    next() {
      // 4. it should return the value as an object {done:.., value :...}
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

// now it works!
for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```

```js
const myIterable = {
  data: [1, 2, 3],
  
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for (const item of myIterable) {
  console.log(item); // Output: 1, 2, 3
}
```

---

### ✅ Explanation

- `Symbol.iterator` is a special key used by the `for...of` loop.
    
- The function returns an **iterator**: an object with a `next()` method.
    
- The `next()` method returns an object with `{ value, done }`.
    

---

### ✅ Use with Spread Operator or Destructuring

Once it's iterable, you can use it like this:

```js
console.log([...myIterable]);     // [1, 2, 3]
const [a, b] = myIterable;        // a = 1, b = 2
```

---

### ✅ Generator-Based Iterable (Simpler)

Using a **generator function** makes iterables cleaner:

```js
const myIterable = {
  data: [1, 2, 3],
  *[Symbol.iterator]() {
    for (let item of this.data) {
      yield item;
    }
  }
};

for (let val of myIterable) {
  console.log(val); // 1, 2, 3
}
```

---

##  Creating Custom Iterables

You can make any object iterable. There are two main ways to do this.

### Method A: The Manual Way (Verbose)

You explicitly return an object with a `next()` method.

```js
const range = {
  from: 1,
  to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    let last = this.to;

    // Return the Iterator object
    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};

console.log([...range]); // Output: [1, 2, 3]
```

### Method B: The Generator Way (Syntactic Sugar)

Generators (`function*`) automatically implement the Iterator Protocol. This is the preferred, modern way.

JavaScript

```
const range = {
  from: 1,
  to: 3,
  // Shorthand for [Symbol.iterator]: function* () { ... }
  *[Symbol.iterator]() {
    for (let i = this.from; i <= this.to; i++) {
      yield i;
    }
  }
};

for (const num of range) {
  console.log(num); // Output: 1, 2, 3
}
```

---

## Advanced Concepts

### Well-Formed Iterators

A "well-formed" iterator is an iterator that is _also_ an iterable. It implies the iterator object has a `[Symbol.iterator]` method that returns `this`.

- **Why?** It allows you to start a loop on an iterator that has already been partially consumed.
    
- **Default:** All built-in iterators (Array iterators, Map iterators) are well-formed.
    

```js
// Example of a well-formed check
const arr = [1, 2];
const it = arr[Symbol.iterator](); // 'it' is the iterator

// 'it' is also iterable because it has Symbol.iterator returning itself
console.log(it[Symbol.iterator]() === it); // true
```

### Infinite Iterables

Because iterators are lazy (they only calculate the _next_ value when asked), you can create infinite sequences without crashing the browser (as long as you don't try to spread `...` them into an array).

JavaScript

```
const infiniteIds = {
  *[Symbol.iterator]() {
    let id = 1;
    while (true) {
      yield id++;
    }
  }
};

// Safe usage with break
for (const id of infiniteIds) {
  if (id > 5) break;
  console.log(id);
}
```

### Async Iterables

For data that arrives asynchronously (e.g., streaming network data), ES2018 introduced **Async Iterators**.

- **Protocol:** `Symbol.asyncIterator`
    
- **Return:** `next()` returns a `Promise` resolving to `{ value, done }`.
    
- **Consumption:** `for await (const item of iterable) { ... }`
    

```js
const asyncRange = {
  async *[Symbol.asyncIterator]() {
    for (let i = 0; i < 3; i++) {
      // Simulate delay
      await new Promise(r => setTimeout(r, 100));
      yield i;
    }
  }
};

(async () => {
  for await (const num of asyncRange) {
    console.log(num); // 0, 1, 2 (with delays)
  }
})();
```

---

##  Summary Cheat Sheet

| **Term**             | **Definition**                   | **Key Feature**               |
| -------------------- | -------------------------------- | ----------------------------- |
| **Iterable**         | Object containing data.          | Has `[Symbol.iterator]()`.    |
| **Iterator**         | Pointer traversing data.         | Has `.next()`.                |
| **IteratorResult**   | The packet returned by next.     | `{ value: any, done: bool }`. |
| **Generator**        | Function that creates iterators. | Uses `function*` and `yield`. |
| **`for...of`**       | Loop that consumes iterables.    | Sync only.                    |
| **`for await...of`** | Loop for async iterables.        | Async (Promises).             |
