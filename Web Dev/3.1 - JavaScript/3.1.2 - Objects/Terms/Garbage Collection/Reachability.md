---
tags:
  - js
  - term
---

## 🧠 What Is Reachability?

In JavaScript, **a value is "reachable"** if it can be accessed somehow from the **roots**. If a value is not reachable, it's considered **garbage** and will be **collected automatically**.

---

### 🌱 What Are "Roots"?

Roots are values that are always accessible, such as:

- Global variables (`window`, `globalThis`)
    
- Local variables inside functions while they are still running
    
- Function arguments and return values
    
- The call stack
    
- Closures that are still referenced
    

---

### 📦 Example: Reachable vs Unreachable

```js
let user = {
  name: "Alice"
};

let admin = user; // now both variables point to the same object

user = null; // the object is still reachable via admin

admin = null; // now the object is unreachable and will be garbage collected
```

The object `{ name: "Alice" }` becomes **unreachable** only when no variable or closure references it anymore.

--- 

### Two references

Now let’s imagine we copied the reference from `user` to `admin`:

```js
// user has a reference to the object
let user = {
  name: "John"
};

let admin = user;
```

Now if we do the same:

```js
user = null;
```

…Then the object is still reachable via `admin` global variable, so it must stay in memory. If we overwrite `admin` too, then it can be removed.

---

### 🧹 Garbage Collector’s Role

- The basic garbage collection algorithm is called “mark-and-sweep”.
- The following “garbage collection” steps are regularly performed:
	- The garbage collector takes roots and “marks” (remembers) them.
	- Then it visits and “marks” all references from them.
	- Then it visits marked objects and marks _their_ references. All visited objects are remembered, so as not to visit the same object twice in the future.
	- …And so on until every reachable (from the roots) references are visited.
	- All objects except marked ones are removed.

![[Pasted image 20250704090216.png|center|400]]

---

### 🧪 Cyclic References?

Even if objects reference each other, they can still be collected if no root references them.

```js
function createCycle() {
  let a = {};
  let b = {};
  a.b = b;
  b.a = a;
}

createCycle(); // `a` and `b` form a cycle, but it's unreachable after function runs
```