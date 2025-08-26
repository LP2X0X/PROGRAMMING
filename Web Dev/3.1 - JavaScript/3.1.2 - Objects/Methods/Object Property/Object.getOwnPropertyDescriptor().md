---
tags: 
 - js
 - object
 - property
 - method
---

- The method [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) allows to query the _full_ information about a property.
</br>
- The syntax is:
	- `obj`: The object to get information from.
	- `propertyName`: The name of the property.
```js
let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName);
```

- The returned value is a so-called “property descriptor” object: it contains the value and all the flags.
```js
let user = {
  name: "John"
};

let descriptor = Object.getOwnPropertyDescriptor(user, 'name');

alert( JSON.stringify(descriptor, null, 2 ) );
/* property descriptor:
{
  "value": "John",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```

- To get all property descriptors at once, we can use the method [Object.getOwnPropertyDescriptors(obj)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors) (which is its plural form).
	```js
	let user = {
	  name: "Alice",
	  get age() { return 30; }
	};
	
	console.log(Object.getOwnPropertyDescriptors(user));
	/*
	{
	  name: {
	    value: "Alice",
	    writable: true,
	    enumerable: true,
	    configurable: true
	  },
	  age: {
	    get: f age,
	    set: undefined,
	    enumerable: true,
	    configurable: true
	  }
	}
	*/
	```