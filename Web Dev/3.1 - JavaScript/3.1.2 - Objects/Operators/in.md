---
tags:
  - js
  - operator
  - fundamental
---

- Reading a non-existing property just returns `undefined`. So we can easily test whether the property exists using the `in` operator:
	```js
	let user = { name: "John", age: 30 };
	
	alert( "age" in user ); // true, user.age exists
	alert( "blabla" in user ); // false, user.blabla doesn't exist
	```