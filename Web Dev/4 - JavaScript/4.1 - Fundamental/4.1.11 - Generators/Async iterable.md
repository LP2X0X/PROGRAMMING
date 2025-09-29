---
tags: 
 - js
 - iterable
 - async
---

# ⚡ Async Iterable in JavaScript

### 1. What It Is

An **async iterable** is any object that implements the special method:

```js
obj[Symbol.asyncIterator]() → returns an async iterator
```

- An **iterator** has a `.next()` method → returns `{ value, done }`.
    
- An **async iterator** has `.next()` → returns a **Promise** resolving to `{ value, done }`.
    

So an **async iterable** is just “something you can loop over asynchronously” with `for await...of`.

---

### 2. Example: Manual Async Iterable

```js
let range = {
  from: 1,
  to: 5,

  [Symbol.asyncIterator]() { // (1)
    return {
      current: this.from,
      last: this.to,

      async next() { // (2)

        // note: we can use "await" inside the async next:
        await new Promise(resolve => setTimeout(resolve, 1000)); // (3)

        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }

// Version which implement generator
//
// this line is same as [Symbol.asyncIterator]: async function*() {
// async *[Symbol.asyncIterator]() {
//   for(let value = this.from; value <= this.to; value++) {
//
//     // make a pause between values, wait for something
//     await new Promise(resolve => setTimeout(resolve, 1000));
//
//     yield value;
//   }
// }
};

(async () => {

  for await (let value of range) { // (4)
    alert(value); // 1,2,3,4,5
  }

})()
```

As we can see, the structure is similar to regular iterators:

1. To make an object asynchronously iterable, it must have a method `Symbol.asyncIterator` `(1)`.
2. This method must return the object with `next()` method returning a promise `(2)`.
3. The `next()` method doesn’t have to be `async`, it may be a regular method returning a promise, but `async` allows us to use `await`, so that’s convenient. Here we just delay for a second `(3)`.
4. To iterate, we use `for await(let value of range)` `(4)`, namely add “await” after “for”. It calls `range[Symbol.asyncIterator]()` once, and then its `next()` for values.

---

### 3. Async Iterable vs Iterable

|Feature|Iterable (`Symbol.iterator`)|Async Iterable (`Symbol.asyncIterator`)|
|---|---|---|
|Loop construct|`for...of`|`for await...of`|
|`.next()` return type|`{ value, done }`|`Promise<{ value, done }>`|
|Suitable for|Arrays, Strings, Maps, Sets|Streams, APIs, timers, async data|

---

### 4. How Async Generator Fits In

When you write:

```js
async function* gen() {
  yield 1;
  yield 2;
}
```

The function automatically creates an **async iterable** object.  
It defines `[Symbol.asyncIterator]` for you internally, so you can use `for await...of` without writing boilerplate.

---

### 5. Real-Life Use Case

Fetching paginated API data chunk by chunk:

```js
async function* fetchUsers(totalPages) {
  for (let p = 1; p <= totalPages; p++) {
    let res = await fetch(`/api/users?page=${p}`);
    yield await res.json();
  }
}

(async () => {
  for await (let users of fetchUsers(3)) {
    console.log(users);
  }
})();
```

This is much more memory-efficient than fetching everything upfront.

---

### The spread syntax `...` doesn’t work asynchronously

Features that require regular, synchronous iterators, don’t work with asynchronous ones.

For instance, a spread syntax won’t work:

```javascript
alert( [...range] ); // Error, no Symbol.iterator
```

That’s natural, as it expects to find `Symbol.iterator`, not `Symbol.asyncIterator`.

It’s also the case for `for..of`: the syntax without `await` needs `Symbol.iterator`.

---

✅ So in summary:

- **Async Iterable** = object with `[Symbol.asyncIterator]`.
    
- Consumed with **`for await...of`**.
    
- Useful for async data streams.
    
- **Async Generators** are the easiest way to _create_ async iterables.
    