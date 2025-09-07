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