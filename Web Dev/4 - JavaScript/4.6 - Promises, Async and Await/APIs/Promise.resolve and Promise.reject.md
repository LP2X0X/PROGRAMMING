---
tags: 
 - js
 - promise
 - api
---

# `Promise.resolve`

### 1. What it does

- Creates a promise that is **resolved** (fulfilled) with the given value.
    

```js
Promise.resolve(42).then(value => console.log(value));
// 42
```

- This is the same as:
```js
let promise = new Promise(resolve => resolve(42));
```

---

### 2. Special Cases

- If you pass in a **promise**, it just returns the same promise:
    
    ```js
    let p = Promise.resolve(99);
    console.log(Promise.resolve(p) === p); // true
    ```
    
- If you pass in a **thenable** (object with `.then`), it assimilates it:
    
    ```js
    let thenable = { then: resolve => resolve("from thenable") };
    Promise.resolve(thenable).then(console.log); 
    // "from thenable"
    ```
    

---

### 3. Example

- The `loadCached` function below fetches a URL and remembers (caches) its content. For future calls with the same URL it immediately gets the previous content from cache, but uses `Promise.resolve` to make a promise of it, so the returned value is always a promise:
	```js
	let cache = new Map();
	
	function loadCached(url) {
	  if (cache.has(url)) {
	    return Promise.resolve(cache.get(url)); // (*)
	  }
	
	  return fetch(url)
	    .then(response => response.text())
	    .then(text => {
	      cache.set(url,text);
	      return text;
	    });
	}
	```
- We can write `loadCached(url).then(…)`, because the function is guaranteed to return a promise. We can always use `.then` after `loadCached`. That’s the purpose of `Promise.resolve` in the line `(*)`.

----

# `Promise.reject`

### 1. What it does

- Creates a promise that is **rejected** with the given reason.
    

```js
Promise.reject("Oops")
  .catch(err => console.log("Caught:", err));
// "Caught: Oops"
```

---

### 2. Key Differences from `resolve`

- Unlike `resolve`, **`reject` does not unwrap thenables**.
    
    ```js
    let thenable = { then: resolve => resolve("hello") };
    Promise.reject(thenable).catch(err => console.log(err));
    // logs the raw thenable object, not "hello"
    ```
    

----

# Comparison

|Method|Behavior|
|---|---|
|`Promise.resolve(x)`|Returns a fulfilled promise with value `x`. If `x` is a promise → returns it. If `x` is thenable → assimilates it.|
|`Promise.reject(x)`|Returns a rejected promise with reason `x`. Never assimilates, always rejects with `x`.|

---

### 3. Quick Use Cases

- **`Promise.resolve`**
    
    - Wrap sync values into promises for uniform async pipelines.
        
    - Assimilate thenables into real promises.
        
- **`Promise.reject`**
    
    - Quickly create a rejected promise (for testing or error handling).
        

---

✅ **Summary**

- `Promise.resolve` is smart: it unwraps promises/thenables into a real settled promise.
    
- `Promise.reject` is simple: it always creates a rejected promise with exactly what you pass.
    