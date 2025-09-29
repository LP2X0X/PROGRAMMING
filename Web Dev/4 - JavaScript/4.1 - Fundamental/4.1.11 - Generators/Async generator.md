---
tags: 
 - js
 - generator
 - async
---

# ⚡ Async Generators

### 1. What They Are

An **async generator** is a function declared with `async function*`.  
It combines:

- `async` → allows `await` inside
    
- `function*` → allows `yield`
    

This means:  
➡️ You can `await` asynchronous operations _before yielding_,  
➡️ and each `yield` produces a value in an **async iterator**.

---

### 2. Example: Async Generator in Action

```js
async function* numbers() {
  yield 1;
  await new Promise(r => setTimeout(r, 1000)); // simulate async work
  yield 2;
  yield 3;
}

(async () => {
  for await (let n of numbers()) {
    console.log(n);
  }
})();
```

✅ Output:

```
1
2 (after 1s delay)
3
```

Notice how `await` inside the generator pauses _just that generator’s execution_ before continuing.

---

### 3. How They Differ from Normal Generators

|Feature|Normal Generator (`function*`)|Async Generator (`async function*`)|
|---|---|---|
|`yield` values|Immediate values|Values or Promises|
|Returned object|Iterator (`next()` → `{value, done}`)|Async Iterator (`next()` → `Promise<{value, done}>`)|
|Loop consumption|`for...of`|`for await...of`|

---

### 4. Practical Use Case: Streaming Data

Imagine fetching paginated API data:

```js
async function* fetchPages(baseUrl, pages) {
  for (let i = 1; i <= pages; i++) {
    let res = await fetch(`${baseUrl}?page=${i}`);
    let data = await res.json();
    yield data;
  }
}

(async () => {
  for await (let page of fetchPages("/api/items", 3)) {
    console.log("Got page:", page);
  }
})();
```

Here, each page is processed as soon as it arrives instead of waiting for all 3.

---

### 5. Async Generator Methods

Just like normal generators, async generators have:

- `.next()`
    
- `.throw()`
    
- `.return()`
    

But each returns a **Promise**. Example:

```js
const g = numbers();

g.next().then(console.log); // { value: 1, done: false }
g.return("done").then(console.log); // { value: "done", done: true }
```

---

✅ In short:

- `async function*` creates **async iterators**.
    
- They let you `await` inside, and you consume them with `for await...of`.
    
- Perfect for **streams, APIs, paginated results, or event sources**.
    