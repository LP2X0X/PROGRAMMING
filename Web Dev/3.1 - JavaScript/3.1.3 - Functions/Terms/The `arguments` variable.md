---
tags: js, term, fundamental
---

- There is also a special [[Array-like Objects|array-like object]] named `arguments` that **contains all arguments** by their index. For instance:
	```js
	function showName() {
	  alert( arguments.length );
	  alert( arguments[0] );
	  alert( arguments[1] );
	
	  // it's iterable
	  // for(let arg of arguments) alert(arg);
	}
	
	// shows: 2, Julius, Caesar
	showName("Julius", "Caesar");
	
	// shows: 1, Ilya, undefined (no second argument)
	showName("Ilya");
	```

````ad-note
- Arrow functions do not have `"arguments"`
- If we access the `arguments` object from an arrow function, it takes them from the outer “normal” function.
	```js
	function f() {
	  let showArg = () => alert(arguments[0]);
	  showArg();
	}
	
	f(1); // 1
````