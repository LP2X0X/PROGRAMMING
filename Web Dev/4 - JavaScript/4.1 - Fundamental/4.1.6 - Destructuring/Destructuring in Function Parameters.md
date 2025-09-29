---
tags:
  - js
  - parameters
  - destructuring
---

- The full syntax is the same as for a destructuring assignment:
	```js
	function({
	  incomingProperty: varName = defaultValue
	  ...
	})
	```

### ✅ **Array Destructuring in Functions**

```javascript
function sum([a, b]) {
  return a + b;
}

console.log(sum([5, 10])); // 15
```

### 📦 **Object Destructuring in Functions**

```javascript
function printUser({ name, age }) {
  console.log(`Name: ${name}, Age: ${age}`);
}

printUser({ name: 'John', age: 25 });
// Output: Name: John, Age: 25
```

### 📌 **Default Values in Function Parameters**

- Please note that such destructuring assumes that `showMenu()` does have an argument. If we want all values by default, then we should specify an empty object:
	```js
	showMenu({}); // ok, all values are default
	
	showMenu(); // this would give an error
	```
- We can fix this by making `{}` the default value for the whole object of parameters:
	```js
	function showMenu({ title = "Menu", width = 100, height = 200 } = {}) {
	  alert( `${title} ${width} ${height}` );
	}
	
	showMenu(); // Menu 100 200
	```
- In the code above, the whole arguments object is `{}` by default, so there’s always something to destructurize.