---
tags: 
 - js
 - object
 - technique
---

- Here's how to do it:
	```js
	let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));
	```

- Normally when we clone an object, we use an assignment to copy properties, like this:
	```js
	for (let key in user) {
	  clone[key] = user[key]
	}
	```

- …But that does not copy flags. So if we want a “better” clone then `Object.defineProperties` is preferred.
- Another difference is that `for..in` ignores symbolic and non-enumerable properties, but `Object.getOwnPropertyDescriptors` returns _all_ property descriptors including symbolic and non-enumerable ones.