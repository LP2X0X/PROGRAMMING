---
tags:
  - js
  - term
  - fundamental
---

## 🧠 Core distinction

**Expression**

- Produces a **value**
    
- Can appear **where a value is expected**
    

**Statement**

- Performs an **action**
    
- Does **not** produce a value
    

---

## 🔹 Expressions

✔ Evaluate to a value  
✔ Can be assigned, logged, passed to functions

**Examples**

```js
42
a + b
foo()
x = 10
condition ? a : b
```

```js
console.log(a + b);   // valid
let y = x = 5;        // assignment expression
```

---

## 🔸 Statements

✘ Do not evaluate to a value  
✘ Cannot be embedded inside expressions

**Examples**

```js
if (x > 0) { }
for (let i = 0; i < 3; i++) { }
const a = 5;
return x;
```

```js
console.log(if (x) {}); // ❌ SyntaxError
```

---

## ⚠️ Common source of confusion

### Assignment vs Declaration

```js
x = 5;        // expression → value is 5
const x = 5;  // statement → no value
```

The `=` operator is an **expression**,  
but `const`, `let`, `var` are **statements**.

---

## 🧩 Expression statements

Some expressions are _used as statements_:

```js
x = 5;
foo();
```

They still have values — JavaScript just **ignores them**.

---

## 🧪 Quick test

Ask:

> “Can I put this inside `console.log(...)`?”

- Yes → **Expression**
    
- No → **Statement**
    

---

## 📌 Key takeaway

> **Expressions produce values.  
> Statements control program flow or declare structure.**