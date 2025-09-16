---
tags: 
 - js
 - term
 - module
---

- A module is just a file. One script is one module. As simple as that.
- Modules can load each other and use special directives `export` and `import` to interchange functionality, call functions of one module from another one:
	- `export` keyword labels variables and functions that should be accessible from outside the current module.
	- `import` allows the import of functionality from other modules.

```ad-note
Modules work only via HTTP(s), not locally!
If you try to open a web-page locally, via `file://` protocol, you’ll find that `import/export` directives don’t work.
```

</br>

- For instance, if we have a file `sayHi.js` exporting a function:
	```js
	// 📁 sayHi.js
	export function sayHi(user) {
	  alert(`Hello, ${user}!`);
	}
	```

- …Then another file may import and use it:
	```js
	// 📁 main.js
	import {sayHi} from './sayHi.js';
	
	alert(sayHi); // function...
	sayHi('John'); // Hello, John!
	```

- The `import` directive loads the module by path `./sayHi.js` relative to the current file, and assigns exported function `sayHi` to the corresponding variable.

---

Here’s a formatted note on the **Core Module Features** section from _“Modules, introduction”_ (javascript.info) — key points, what they mean, and gotchas.

---

## 📋 Core Module Features 

These are features that all JavaScript modules share (browser + server), different from “old-style scripts”.

---

### 1. Always `"use strict"`

- Modules always run in **strict mode** by default. 
    
- Behavior changes under strict mode, e.g.:
    
    - Using undeclared variables throws error.
        
    - `this` at module top-level is `undefined` instead of the global object.
        

```html
<script type="module">
  a = 5; // error
</script>
```

**Gotchas:**

- If you expect sloppy-mode behavior (global leaking, etc.), it won’t work.
    
- Some older code (or libraries) might assume non-strict mode.
    

---

### 2. Module-level scope (no automatic global leakage)

- Top-level variables/functions in a module are **private to that module**.
    
- To share something, you must `export` it. To use it elsewhere, you must `import`.
    

**Gotchas:**

- Modules do _not_ see each other’s globals unless explicitly exported/imported.
    
- If you load multiple `<script type="module">` tags in HTML, they each get their own module scope (don’t share top-level vars).
    

---

### 3. Module code evaluated only once

- If a module is imported from multiple places, its body runs just **once**, at the first import. Afterwards, its exported bindings are reused (passed to all further importers). 
    
- So side effects defined at the top level happen only once. This is convenience because you can configure a module **once** (e.g., setting API credentials, database connections, feature flags) and then every part of your app can use it without having to pass the configuration around.
	```js
	// 📁 1.js
	import {admin} from './admin.js';
	admin.name = "Pete";
	
	// 📁 2.js
	import {admin} from './admin.js';
	alert(admin.name); // Pete
	
	// Both 1.js and 2.js reference the same admin object
	// Changes made in 1.js are visible in 2.js
	```

```ad-note
There’s a rule: top-level module code should be used for initialization, creation of module-specific internal data structures. If we need to make something callable multiple times – we should export it as a function.
```

**Gotchas:**

- If you expect side-effects in each import, you’ll be surprised.
    
- Shared mutable exports = shared state among all importers. Changes in one place are visible elsewhere.
    

---

### 4. `import.meta`

- Each module gets `import.meta`, an object with metadata about the module.
    
- In browsers, it contains like `url` (the module’s URL).
    

**Gotchas:**

- Behavior of `import.meta` differs by environment (browser vs Node.js).
    
- It’s read-only metadata; can’t change much inside it.
    

---

### 5. Top-level `this` is `undefined`

- In module code (at top level), `this` is **not** the global object; it’s `undefined`. 
    

**Gotchas:**

- If code assumes `this` refers to `window` (browser) or `global` (Node), it’ll break.
    
- Minor, but can trip up when adapting non-module scripts.
    

---

### 6. Module scripts are deferred

- `<script type="module">` is implicitly **deferred**:
    
    - They load in parallel.
        
    - They run after the HTML is parsed. 
        
- Inline module scripts also follow deferred behavior.
    

**Gotchas:**

- Since scripts run after HTML, code that tries to access DOM elements defined later in HTML (before script) will work more often than with non-deferred scripts.
    
- Ordering among module scripts is preserved.
    

---

## 🔍 Why These Features Matter

- They provide **better encapsulation**, avoiding global namespace pollution.
    
- Enable **modularity and reuse** (export/import).
    
- Predictable execution (only run once, strict mode).
    
- More secure / fewer traps than older script model.
    
