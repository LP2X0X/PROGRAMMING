---
tags: js, distinguish
---

## 🧠 **Closures vs Function Properties** in JavaScript

|Aspect|**Closure**|**Function Property**|
|---|---|---|
|**Definition**|Function that “remembers” variables from its lexical scope even after the outer function has returned.|Using the fact that functions are objects to store state as a property.|
|**Where data is stored**|In the **Lexical Environment** of the outer function.|In the **function object** itself (`function.property`).|
|**Encapsulation**|✅ **Private** – cannot be accessed from outside unless exposed.|❌ **Public** – the property is accessible and mutable externally.|
|**Usage**|When you want to hide state and keep data safe from external access.|When you don’t mind exposing the state (or even want that).|
|**Memory**|Creates a new scope for each function call.|Stores on one shared function object – simpler, more lightweight.|
|**Example**|See below 👇|See below 👇|

---

## 🔐 **Closure Example** – _Private state_

```js
function makeCounter() {
  let count = 0;
  return function() {
    return count++;
  };
}

let counter = makeCounter();
console.log(counter()); // 0
console.log(counter()); // 1
// count is not accessible here
```

- ✅ `count` is private
    
- 🚫 You **can’t access** or modify `count` from the outside
    

---

## 🧮 **Function Property Example** – _Public state_

```js
function makeCounter() {
  function counter() {
    return counter.count++;
  }
  counter.count = 0;
  return counter;
}

let counter = makeCounter();
console.log(counter()); // 0
console.log(counter()); // 1
console.log(counter.count); // 2
counter.count = 100; // You can even modify it!
```

- ❗ `count` is stored **on the function itself**
    
- ✅ You **can access or modify** `counter.count` from outside
    

---

## 🟨 Summary

|Use Closure When:|Use Function Property When:|
|---|---|
|You want to **hide** internal variables|You want a **simple, exposed counter**|
|You need **better encapsulation**|You prefer **quick functional state**|
|You’re dealing with multiple instances with **isolated state**|You’re okay with **one shared instance**|