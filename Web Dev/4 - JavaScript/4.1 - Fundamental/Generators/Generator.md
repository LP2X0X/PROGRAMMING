---
tags: 
 - js
 - generator
---

## 🔹 What is a Generator?

A **generator** is a special kind of <u>function</u> that can **pause execution and later resume**, while remembering its context (variables, state, etc.). It is a [[Iterables|iterable]].

You declare one with the `function*` syntax:

```js
function* myGenerator() {
  yield 1;
  yield 2;
  yield 3;
}
```

Notice the `*` after `function`.

```ad-note
`function* f(…)` or `function *f(…)`?

Both syntaxes are correct.

But usually the first syntax is preferred, as the star `*` denotes that it’s a generator function, it describes the kind, not the name, so it should stick with the `function` keyword.
```

---

## 🔹 How do they work?

When you call a generator function, it doesn’t run right away. Instead, it returns a **generator object** (an iterator).

```js
const gen = myGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

- Each call to `.next()` resumes execution until the next `yield`.
    
- The `yield` keyword _pauses_ the function and produces a value.
    
- Once the function runs out of `yield`s (or hits `return`), `done: true`.
    

---

## 🔹 The result of "next"

The result of `next()` is always an object with two properties:

- `value`: the yielded value.
- `done`: `true` if the function code has finished, otherwise `false`.

For instance, here we create the generator and get its first yielded value:
```js
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

let one = generator.next();

alert(JSON.stringify(one)); // {value: 1, done: false}
```

---

### 🔄 Generator Composition

**Definition**: Generator composition is when one generator delegates part of its yield sequence to another generator using the `yield*` syntax.

Example

```js
function* innerGen() {
  yield 1;
  yield 2;
}

function* outerGen() {
  yield 0;
  yield* innerGen(); // delegate to innerGen
  yield 3;
}

for (const value of outerGen()) {
  console.log(value);
}
// Output: 0, 1, 2, 3
```

#### How It Works

- `yield* otherGen()`:
    
    - Iterates over `otherGen`.
        
    - Yields each value **as if it were part of the current generator**.
        
- Useful for splitting complex sequences into smaller reusable generators.
    

#### Gotchas ⚠️

1. `yield*` only works with **iterables** (not just generators).
    
    ```js
    function* g() {
      yield* [1, 2, 3]; // works with arrays too
    }
    ```
    
2. If the delegated generator returns a value (via `return`), you can capture it:
    
    ```js
    function* inner() {
      yield 1;
      return "done";
    }
    
    function* outer() {
      let result = yield* inner();
      console.log(result); // "done"
    }
    
    [...outer()];
    ```
    

---

## 🔹 Yield as a "two-way street"

You can also **send values back into** a generator:

```js
function* counter() {
  let x = yield "First stop";
  yield x * 2;
}

const gen = counter();
console.log(gen.next());     // { value: "First stop", done: false }
console.log(gen.next(10));   // { value: 20, done: false }
console.log(gen.next());     // { value: undefined, done: true }
```

```ad-note
That’s because `yield` is a two-way street: it not only returns the result to the outside, but also can pass the value inside the generator.

To do so, we should call `generator.next(arg)`, with an argument. ***That argument becomes the result of `yield`.***
```

Here:

- The first `next()` starts until the first `yield`.
    
- The second `next(10)` sends `10` back as the result of the first `yield`.
    

---

## `generator.throw(error)`

- Purpose: *Inject* an **error** into the generator at the point of the current `yield`.
    
- If the generator has a `try..catch` around the `yield`, the error can be caught inside the generator.
    
- If not caught, the error bubbles out like a normal `throw`.
    

```js
function* gen() {
  try {
    yield 1;
    yield 2;
  } catch (err) {
    console.log("Caught inside generator:", err.message);
  }
  yield 3;
}

const g = gen();

console.log(g.next());       // { value: 1, done: false }
console.log(g.throw(new Error("Boom!"))); 
// caught inside generator -> logs "Caught inside generator: Boom!"
// then continues to yield 3
```

➡️ **Gotcha:** If you throw when the generator is already done, the error will propagate outside immediately.

---

## `generator.return(value)`

- Purpose: Forcefully **finish** the generator early, optionally providing a final return value.
    
- It immediately marks the generator as `done: true`.
    
- A `finally` block inside the generator will still execute.
    

```js
function* gen() {
  try {
    yield 1;
    yield 2;
  } finally {
    console.log("Cleanup before finishing");
  }
}

const g = gen();

console.log(g.next());          // { value: 1, done: false }
console.log(g.return(99));      // "Cleanup before finishing"
// -> { value: 99, done: true }
console.log(g.next());          // { value: undefined, done: true }
```

➡️ **Gotcha:**

- `.return()` does **not** resume yielding — it finalizes.
    
- Useful for cleanup, like closing connections or releasing resources.
    

---

## 🔹 Use Cases

1. **Lazy evaluation / data streams**
    
    ```js
    function* infiniteNumbers() {
      let n = 0;
      while (true) yield n++;
    }
    
    const nums = infiniteNumbers();
    console.log(nums.next().value); // 0
    console.log(nums.next().value); // 1
    ```
    
    → Efficient because values are generated on demand.
    
2. **Custom iteration**
    
    ```js
    const range = {
      from: 1,
      to: 5,
      *[Symbol.iterator]() {
        for (let i = this.from; i <= this.to; i++) yield i;
      }
    };
    
    for (let num of range) {
      console.log(num); // 1..5
    }
    ```
    
    → Generators integrate naturally with `for...of`.
    
3. **Async control flow (before async/await existed)**  
    Libraries like `co` used generators to write async code that _looked synchronous_.
    

---

## 🔹 Gotchas & Notes

- A generator runs **synchronously until it hits a `yield`**.
    
- `.next()` is how you control when it resumes.
    
- Once a generator finishes (`done: true`), you can’t restart it — you must create a new generator.
    
- You can throw into a generator with `.throw(error)` to simulate an error inside it.
    

---

✨ **Big picture:**

- A normal function → runs start to finish, no stopping.
    
- A generator function → can pause (`yield`), resume (`next`), and exchange values back and forth.
    