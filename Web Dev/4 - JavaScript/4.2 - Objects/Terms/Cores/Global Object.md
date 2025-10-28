---
tags:
  - js
  - object
  - fundamental
---

- In JavaScript, the **global object** is a special object that is automatically created by the environment and serves as the top-level container for all global variables, functions, and built-in objects.

- It has two roles:
	1. First, it is a global object for JavaScript code.
	2. Second, it represents the “browser window” and provides methods to control it.

- In a browser, global functions and variables declared with `var` (not `let/const`!) become the property of the global object:
	```js
	var gVar = 5;
	
	alert(window.gVar); // 5 (became a property of the global object)
	```
- If a value is so important that you’d like to make it available globally, write it directly as a property:
	```js
	// make current user information global, to let all scripts access it
	window.currentUser = {
	  name: "John"
	};
	
	// somewhere else in code
	alert(currentUser.name);  // John
	
	// or, if we have a local variable with the name "currentUser"
	// get it from window explicitly (safe!)
	alert(window.currentUser.name); // John
	```

---

### 🧠 Key Points About the Global Object:

|Environment|Global Object Name|
|---|---|
|Browser|`window`|
|Web Workers|`self`|
|Node.js|`global`|
|Universal (ES2020+)|`globalThis`|

---

### 🌍 What Does It Contain?

- Built-in constructors: `Object`, `Array`, `Function`, `Date`, etc.
    
- Global functions: `parseInt()`, `setTimeout()`, `isNaN()`, etc.
    
- Global variables: any variable declared **without `let`, `const`, or `var`** inside global scope.
    
- In browsers: also contains everything defined in the global HTML document like `document`, `location`, etc.
    

---

### 🧪 Example:

```js
// In a browser
console.log(window === globalThis); // true
console.log(globalThis.setTimeout); // function setTimeout() { ... }

// Adding a global variable (not recommended)
foo = 42; // Implicit global
console.log(globalThis.foo); // 42
```

---

### ☝️Using for polyfills
- We use the global object to test for support of modern language features.
- For instance, test if a built-in `Promise` object exists (it doesn’t in really old browsers):
	```js
	if (!window.Promise) {
	  alert("Your browser is really old!");
	}
	```
- If there’s none (say, we’re in an old browser), we can create “polyfills”: add functions that are not supported by the environment, but exist in the modern standard.
	```js
	if (!window.Promise) {
	  window.Promise = ... // custom implementation of the modern language feature
	}
	```

---

### 🚫 Best Practice:

Avoid relying on the global object directly. Use modular code (with `import`/`export`, or IIFEs) and avoid polluting the global scope to prevent conflicts and bugs.