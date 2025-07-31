---
tags: js, nested, destructuring
---

- If an object or an array contains other nested objects and arrays, we can use more complex left-side patterns to extract deeper portions:
	- Object:
		```js
		let options = {
		  size: {
		    width: 100,
		    height: 200
		  },
		  items: ["Cake", "Donut"],
		  extra: true
		};
		
		// destructuring assignment split in multiple lines for clarity
		let {
		  size: { // put size here
		    width,
		    height
		  },
		  items: [item1, item2], // assign items here
		  title = "Menu" // not present in the object (default value is used)
		} = options;
		
		alert(title);  // Menu
		alert(width);  // 100
		alert(height); // 200
		alert(item1);  // Cake
		alert(item2);  // Donut
		```
	- Array:
		```js
		const matrix = [[1, 2], [3, 4]];
		const [[a, b], [c, d]] = matrix;
		
		console.log(a, b, c, d); // 1 2 3 4
		```