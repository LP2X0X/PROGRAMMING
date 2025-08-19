---
tags:
  - js
  - function
  - fundamental
---

- The call `structuredClone(object)` clones the `object` with all nested properties.
	```js
	let user = {
	  name: "John",
	  sizes: {
	    height: 182,
	    width: 50
	  }
	};
	
	let clone = structuredClone(user);
	
	alert( user.sizes === clone.sizes ); // false, different objects
	
	// user and clone are totally unrelated now
	user.sizes.width = 60;    // change a property from one place
	alert(clone.sizes.width); // 50, not related
	```
- The `structuredClone` method can clone most data types, such as objects, arrays, primitive values.
	```ad-note
	Function properties aren’t supported.
	```