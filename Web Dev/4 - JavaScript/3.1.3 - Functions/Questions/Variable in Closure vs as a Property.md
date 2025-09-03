---
tags: 
 - js 
 - question 
 - advance
---

### 1. **[[Closure|Closure]] variables**

```js
function makeCounter() {
  let count = 0;   // <-- closure variable

  return function() {
    return ++count;
  };
}

let counter = makeCounter();
counter(); // 1
counter(); // 2
```

- `count` is private, only accessible inside the returned function.
    
- Good for **state that belongs to this one instance**.
    
- Prevents accidental overwrites because no outside code can touch `count`.
    

---

### 2. **[[Web Dev/4 - JavaScript/3.1.3 - Functions/Terms/Function|Function]] properties**

```js
function counter() {
  return ++counter.count;
}
counter.count = 0;

counter(); // 1
counter(); // 2
```

- Here, state is stored as a property on the function object.
    
- Anyone can read or modify it (`counter.count = 999`).
    
- Good if you _want_ that state to be **visible and tweakable externally**, or if multiple functions need to share state.
    

---

### 🔑 Key difference

- **Closure variable** → hidden, private, encapsulated.
    
- **Function property** → public, exposed, part of the function’s "API".
    

---

### Example in context: `debounce`

With **closure variable**:

```js
function debounce(f, ms) {
  let timeoutId;  // private, no one else can touch it

  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => f.apply(this, args), ms);
  };
}
```

- `timeoutId` is safe inside the closure.
    
- Outside code can’t accidentally mess with it.
    

With **function property**:

```js
function debounce(f, ms) {
  function wrapper(...args) {
    clearTimeout(wrapper.timeoutId);
    wrapper.timeoutId = setTimeout(() => f.apply(this, args), ms);
  }
  return wrapper;
}
```

- Here `wrapper.timeoutId` is exposed.
    
- You could do things like:
    
    ```js
    let fn = debounce(console.log, 500);
    console.log(fn.timeoutId); // you can inspect or even override it
    ```
    

This is sometimes useful for **testing or debugging**, but riskier because other code could break it.

---

### ⚡ Rule of thumb

- Use a **closure variable** when the state is internal implementation detail (like `timeoutId` in debounce).
    
- Use a **function property** if you want that state to be **part of the API**, accessible or configurable externally.
    