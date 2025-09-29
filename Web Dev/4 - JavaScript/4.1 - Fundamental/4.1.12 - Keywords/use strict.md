---
tags:
  - js
  - keyword
  - mode
---

- When ECMAScript 5 (ES5) appeared, It added new features to the language and modified some of the existing ones. To keep the old code working, most such modifications are off by default. You need to explicitly enable them with a special directive: `"use strict"`.
- The directive looks like a string: `"use strict"` or `'use strict'`. When it is located at the top of a script, the whole script works the “modern” way.

```javascript
"use strict"; // Must be on top to take affect

// this code works the modern way
...
```

- There is no directive like `"no use strict"` that reverts the engine to old behavior.

---

# ⚡ What `"use strict"` Does

### 1. Prevents the use of undeclared variables

```js
"use strict";
x = 10; // ❌ ReferenceError
```

Without strict mode, this would implicitly create a global variable.

---

### 2. Makes assignments to non-writable properties throw errors

```js
"use strict";
const obj = {};
Object.defineProperty(obj, "a", { value: 1, writable: false });

obj.a = 2; // ❌ TypeError
```

---

### 3. Prevents deletion of undeletable properties

```js
"use strict";
delete Object.prototype; // ❌ TypeError
```

---

### 4. Eliminates `this` binding to global object

```js
"use strict";
function f() {
  console.log(this); 
}
f(); // undefined (instead of window/global)
```

---

### 5. Forbids duplicate parameter names

```js
"use strict";
function sum(a, a, c) { // ❌ SyntaxError
  return a + a + c;
}
```

---

### 6. Reserved keywords

Future keywords (like `implements`, `interface`, `package`, `private`, etc.) can’t be used as identifiers.

```js
"use strict";
let package = 1; // ❌ SyntaxError
```

---

### 7. Disables `with`

```js
"use strict";
with (Math) { // ❌ SyntaxError
  console.log(sin(2));
}
```

---

### 8. Safer eval

`eval` in strict mode does **not** introduce variables into the surrounding scope.

```js
"use strict";
eval("var x = 2;");
console.log(typeof x); // "undefined"
```

---

# ✅ Why Use It

- Catches silent errors.
    
- Makes code more predictable and secure.
    
- Encourages cleaner, modern JavaScript.
    
- Many ES6+ features assume strict mode automatically (e.g., classes, modules).
    