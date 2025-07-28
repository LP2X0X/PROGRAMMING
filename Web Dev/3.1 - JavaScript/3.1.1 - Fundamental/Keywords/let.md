---
tags: js, keyword, fundamental
---

- **let** is a keyword which help create a [[Web Dev/3.1 - JavaScript/3.1.1 - Fundamental/Terms/Miscellaneous/Variable|variable]] in JavaScript.

---
 
 ### Why let is preferred over var:

#### **1. Block Scope (✅ Better scoping rules)**

- var is **function-scoped** — it ignores blocks like if, for, etc.
    
- let is **block-scoped** — it respects curly braces {}.
    

**Example:**

```js
if (true) {
  var a = 1;
  let b = 2;
}

console.log(a); // 1 ✅ (but this might be unexpected!)
console.log(b); // ❌ ReferenceError
```

---

#### **2. No Hoisting Surprises**

- Both var and let are [[Hoist|hoisted]], but:
    - var is initialized as undefined.
    - let is not initialized, and using it before declaration gives a **ReferenceError**.

**Example:**

```js
console.log(a); // undefined
var a = 10;

console.log(b); // ❌ ReferenceError
let b = 20;
```

---

#### **3. No Redeclaration in the Same Scope**

- var allows you to declare the same variable more than once in the same scope (which can cause bugs).
    
- let does **not** allow this.
    

**Example:**

```js
var x = 5;
var x = 10; // ✅ Allowed

let y = 5;
let y = 10; // ❌ SyntaxError
```

---

#### **4. Cleaner Code in Loops**

When using loops like for, let gives each iteration its own scope, which prevents issues with closures.

**Example:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 3, 3, 3

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 0, 1, 2 ✅
```

---

#### **✅ Summary: Use let (or const) Instead of var**

|**Feature**|var|let|
|---|---|---|
|Scope|Function|Block|
|Hoisting behavior|Yes, as undefined|Yes, but in TDZ (Temporal Dead Zone)|
|Redeclaration allowed|Yes|No|
|Common bugs|Many|Fewer|

---

- In modern JavaScript, let and const have mostly replaced var. Use:
	- let for variables that will change
	- const for variables that won’t change
