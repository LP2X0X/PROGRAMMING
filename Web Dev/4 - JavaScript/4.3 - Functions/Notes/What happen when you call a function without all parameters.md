---
tags: 
 - js
 - note
---


In JS, **function parameters default to `undefined`** if you don’t pass them.

Example:

```js
function greet(name, age) {
  console.log(`Name: ${name}, Age: ${age}`);
}

greet("Alice");   // Only 1 argument passed
```

👉 Output:

```
Name: Alice, Age: undefined
```

---

### 🔹 Behavior

- Missing arguments → become `undefined`.
    
- No error is thrown (JS doesn’t enforce arity like some languages).
    
- You can check for `undefined` to handle missing arguments.
    

---

### 🔹 Ways to handle missing parameters

1. **Default parameters**
    

```js
function greet(name, age = 18) {
  console.log(`Name: ${name}, Age: ${age}`);
}

greet("Alice"); // "Name: Alice, Age: 18"
```

2. **Manual check**
    

```js
function greet(name, age) {
  if (age === undefined) age = 18;
  console.log(`Name: ${name}, Age: ${age}`);
}
```

3. **Rest parameters** (variable-length args)
    

```js
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum());        // 0
```

---

### 🔹 Contrast with other languages

- **C# / Java** → compiler error if you don’t supply required params (unless they have defaults/overloads).
    
- **Python** → `TypeError` if a required param is missing.
    
- **JS** → permissive, missing args become `undefined`.
    

---

⚡ **Gotcha**:  
If you call with **more arguments than parameters**, the extras are ignored **unless you use `arguments` or rest (`...`)**.

```js
function test(a, b) {
  console.log(a, b);
}

test(1, 2, 3, 4); // 1 2 (extra args ignored)
```