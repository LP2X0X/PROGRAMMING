---
tags: 
 - js
 - fundamental
 - type
 - reference
---

## 1. First, what are “compound types”?

In JS, types are usually split into two broad groups:

- **Primitive types** (value types):  
    `string`, `number`, `bigint`, `boolean`, `symbol`, `null`, `undefined`
    
- **Compound types** (non-primitives, i.e. objects):  
    `Object`, `Array`, `Function`, `Date`, `RegExp`, `Map`, `Set`, etc.  
    (basically: everything that’s not a primitive)
    

---

## 2. Reference vs Value

- **Primitive values** → stored directly, assigned/copied **by value**.
    
- **Objects (compound)** → stored as references, assigned/copied **by reference** (well, the reference itself is copied, not the object).
    

Example:

```js
let x = 10;
let y = x;
y = 20;

console.log(x); // 10 (separate copies)
```

But with objects:

```js
let a = { value: 1 };
let b = a;   // copy the reference
b.value = 99;

console.log(a.value); // 99 (same object)
```

---

## 3. So, answer:

✅ Yes — in JavaScript, **all compound types (non-primitives) are reference types**.

That includes arrays, objects, functions, dates, maps, sets, regexes, typed arrays, etc.

---

## 4. One subtle thing

When you pass an object to a function, you’re **passing the reference by value** (not pass-by-reference like in C++).

```js
function change(obj) {
  obj.value = 42; // affects original object
  obj = {};       // reassigning the parameter (local only)
}

let o = { value: 1 };
change(o);

console.log(o); // { value: 42 }
```

So:

- Object properties can be mutated inside the function.
    
- But reassigning the parameter itself doesn’t affect the outside variable.
    