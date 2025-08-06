---
tags: js, method
---

- To decode a JSON-string, we need another method named JSON.parse:
	```js
	let value = JSON.parse(str[, reviver]);
	```
	- str: JSON-string to parse.
	- reviver: Optional function(key,value) that will be called for each `(key, value)` pair and can transform the value. 

- Example:
	```js
	// stringified array
	let numbers = "[0, 1, 2, 3]";
	
	numbers = JSON.parse(numbers);
	
	alert( numbers[1] ); // 1
	```

- Or for nested objects:
	```js
	let userData = '{ "name": "John", "age": 35, "isAdmin": false, "friends": [0,1,2,3] }';
	
	let user = JSON.parse(userData);
	
	alert( user.friends[1] ); // 1
	```

```ad-note
The json file must have no mistake and *no comment*!
```js
let json = `{
  name: "John",                     // mistake: property name without quotes
  "surname": 'Smith',               // mistake: single quotes in value (must be double)
  'isAdmin': false                  // mistake: single quotes in key (must be double)
  "birthday": new Date(2000, 2, 3), // mistake: no "new" is allowed, only bare values
  "friends": [0,1,2,3]              // here all fine
}`;
````