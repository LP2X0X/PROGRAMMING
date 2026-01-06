---
tags:
  - js
  - term
  - fundamental
---

- Functions can be multiply-nested. For example:
	- A function (`A`) contains a function (`B`), which itself contains a function (`C`).
	- Both functions `B` and `C` form closures here. So, `B` can access `A`, and `C` can access `B`.
	- In addition, since `C` can access `B` which can access `A`, `C` can also access `A`.

- Thus, the closures can contain multiple scopes; they recursively contain the scope of the functions containing it. This is called _scope chaining_.
	```js
	function A(x) {
	  function B(y) {
	    function C(z) {
	      console.log(x + y + z);
	    }
	    C(3);
	  }
	  B(2);
	}
	A(1); // Logs 6 (which is 1 + 2 + 3)
	```

---

### Example

Consider this:

```js
function b() {
	console.log(myVar);
}

function a() {
	var myVar = 2;
	b();
}

var myVar = 1;
a();
```

Looking at this, what do you think get logged to the console?

````ad-Answer
The answer is 1. Why?
Two separate concepts are involved here: **[[Web Dev/4 - JavaScript/4.3 - Functions/4.3.5 - Scopes/Execution Context|execution context]]** and **[[Lexical Environment|lexical environment]]**:

### 1 - The Execution Context
This is a common source of confusion.  
The execution context determines **when and how functions are invoked** (via the call stack), **not** what variables a function can access.

Even though `b()` is called from inside `a()`, that does **not** mean `b` can access `a`’s local variables.

### 2 - The Lexical Environment

Variable lookup is governed by the **lexical environment**, not the call stack.

When `console.log(myVar)` runs inside `b()`:

1. JavaScript looks for `myVar` inside `b`’s own scope → not found
    
2. It then looks in `b`’s **outer (or reference) lexical environment**
    

The critical point:

- The outer lexical environment is determined by **where the function is defined**, not where it is called.
    
- Function `b` is defined in the **global scope**, so its outer lexical environment is the **global execution context**.
    

> In other words, the outer lexcial environment here is not the one above it in the call stack (or execution stack), its the one that sits physically (or lexically) outside of the function b, in this case is the lexical environment of the global execution context which contains myVar = 1 initialization.

The global scope contains:

`var myVar = 1;`

So `b()` logs **`1`**.

---

### Key Takeaway

> **Scope is lexical, not dynamic.**  
> JavaScript resolves variables based on where functions are written, not where they are called.
````