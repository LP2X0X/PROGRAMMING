---
tags: 
 - js
 - function
 - property
 - custom
---

Adding properties to functions in JavaScript may seem unusual at first, but it’s actually a powerful feature that lets you **store additional information or behavior directly on a function object**. Here’s a detailed breakdown of why and when you might do it:

---

### 1. **Functions in JS are objects**

In JavaScript, functions are _first-class objects_. This means you can:

- Assign them to variables.
    
- Pass them as arguments.
    
- Return them from other functions.
    
- **Attach properties to them**, just like any object.
    

Example:

```js
function greet(name) {
  console.log(`Hello, ${name}!`);
}

greet.language = "English"; // adding a property
console.log(greet.language); // "English"
```

---

### 2. **Use case: storing metadata**

Sometimes a function needs some additional information about itself:

```js
function fetchData() {
  // fetch something
}
fetchData.url = "https://api.example.com/data";
fetchData.method = "GET";

console.log(fetchData.url); // "https://api.example.com/data"
```

This keeps the metadata close to the function that uses it, instead of creating a separate object.

---

### 3. **Use case: caching / memoization**

Functions can remember previously computed results by storing them on themselves:

```js
function factorial(n) {
  if (!factorial.cache) factorial.cache = {}; // initialize cache
  if (factorial.cache[n]) return factorial.cache[n];

  if (n <= 1) return 1;
  let result = n * factorial(n - 1);
  factorial.cache[n] = result; // save to cache
  return result;
}

console.log(factorial(5)); // 120
console.log(factorial.cache); // { '2': 2, '3': 6, '4': 24, '5': 120 }
```

---

### 4. **Use case: function-based configuration**

Some libraries use function properties to store configuration or behavior flags:

```js
function doSomething() {}
doSomething.enabled = true;

if (doSomething.enabled) {
  console.log("Function is enabled!");
}
```

---

### 5. **Use case: implementing singletons**

You can use a property to store state that persists across calls:

```js
function counter() {
  counter.count = (counter.count || 0) + 1;
  return counter.count;
}

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

---

✅ **Summary**: Adding properties to functions is useful when you want a **function to carry its own data or metadata**, like caching results, storing configuration, or tracking state. It’s a neat way to combine **behavior and data** in one place.