---
tags: 
 - web dev
---

## 🔹 The Core Idea

In JavaScript, **functions are objects**. When you pass a function around (e.g., as a callback), you are only passing **the function itself**, not its "owner" (the object it was originally attached to).

That’s why the **`this` binding is lost**.

---

## 🔹 Example: Losing `this`

```js
const user = {
  name: "Alice",
  sayHi() {
    console.log("Hi, " + this.name);
  }
};

setTimeout(user.sayHi, 1000); // ❌ Problem
```

👉 After 1 second, this prints:

```
Hi, undefined
```

Why?

- `user.sayHi` is passed as a _bare function reference_.
    
- `setTimeout` calls it without an object context.
    
- Inside the call, `this` is `undefined` (in strict mode) or `window` (non-strict mode).
    

---

## 🔹 How `this` Binding Works

`this` is **not bound at function definition**. It’s bound at **call time**, depending on _how the function is called_:

- `obj.method()` → `this = obj`
    
- `func()` → `this = undefined` (or `window` in sloppy mode)
    
- `func.call(obj)` → `this = obj`
    

So when you pass `user.sayHi`, you’ve stripped away the fact it belonged to `user`.

---

## 🔹 Ways to Fix It

### 1. Use `bind`

```js
setTimeout(user.sayHi.bind(user), 1000);
```

Here, `bind` permanently fixes `this` to `user`.

---

### 2. Wrap in a function

```js
setTimeout(() => user.sayHi(), 1000);
```

Here, the arrow function closes over `user` and calls it correctly.

---

### 3. Arrow functions in methods (class syntax)

```js
class User {
  name = "Alice";
  sayHi = () => {
    console.log("Hi, " + this.name);
  };
}

const u = new User();
setTimeout(u.sayHi, 1000); // works, arrow keeps lexical `this`
```

---

✅ **Summary**:  
When you _pass a method as a parameter_, JavaScript only passes the **function reference**, not its original object binding. Since `this` is determined **at call time**, it defaults to `undefined` (strict mode) unless you explicitly bind it, wrap it, or use an arrow function to preserve the context.