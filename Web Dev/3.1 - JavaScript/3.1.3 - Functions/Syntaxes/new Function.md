---
tags: function, syntax, advance
---

- The syntax for creating a function:
	```js
	let func = new Function ([arg1, arg2, ...argN], functionBody);
	```

- The function is created with the arguments `arg1...argN` and the given `functionBody`. Here’s a function with two arguments:
	```js
	let sum = new Function('a', 'b', 'return a + b');
	
	alert( sum(1, 2) ); // 3
	```

- And here there’s a function without arguments, with only the function body:
	```js
	let sayHi = new Function('alert("Hello")');
	
	sayHi(); // Hello
	```

- The major difference from other ways we’ve seen is that the function is created literally from a string, that is passed at run time.
- All previous declarations required us, programmers, to write the function code in the script.
- But `new Function` allows to turn any string into a function. For example, we can receive a new function from a server and then execute it:
	```js
	let str = ... receive the code from a server dynamically ...
	
	let func = new Function(str);
	func();
	```

- It is used in very specific cases, like when we receive code from a server, or to dynamically compile a function from a template, in complex web-applications.

--- 

### No Closure over Outer Scope

- Unlike regular functions, a function created via `new Function` **does not close over its surrounding lexical environment**.
- Instead, its `[[Environment]]` is always the **global scope**, regardless of where or when it's created:
	```js
	function getFunc() {
	  let value = 'test';
	  let func = new Function('alert(value)');
	  return func;
	}
	getFunc()(); // ReferenceError: value is not defined
	```
- The above code fails because the dynamically created function cannot access `value` from the outer scope.
=> **Avoid Problems with [[Minifier|Minifiers]]**: In production builds, local variable names might be compressed or renamed. If `new Function` could access these renamed locals, it might break due to mismatches. Restricting it to the global scope prevents such issues.