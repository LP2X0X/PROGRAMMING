---
tags: js, term, fundamental
---

- New language features may include not only syntax constructs and operators, but also built-in functions.
- A script that updates/adds new functions is called “polyfill”. It “fills in” the gap and adds missing implementations.
	```js
	// This is a pollyfills
	if (!Math.trunc) { // if no such function
	  // implement it
	  Math.trunc = function(number) {
	    // Math.ceil and Math.floor exist even in ancient JavaScript engines
	    // they are covered later in the tutorial
	    return number < 0 ? Math.ceil(number) : Math.floor(number);
	  };
	}
	```