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