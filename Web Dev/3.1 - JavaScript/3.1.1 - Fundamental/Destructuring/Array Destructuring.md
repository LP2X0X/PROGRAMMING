---
tags: js, destructuring, array, fundamental
---

- Here’s an example of how an array is destructured into variables:
	```javascript
	// we have an array with a name and surname
	let arr = ["John", "Smith"]
	
	// destructuring assignment
	// sets firstName = arr[0]
	// and surname = arr[1]
	let [firstName, surname] = arr;
	
	alert(firstName); // John
	alert(surname);  // Smith
	```
- Now we can work with variables instead of array members.

---

#### Ignore elements using commas

- Unwanted elements of the array can also be thrown away via an extra comma:
	```javascript
	// second element is not needed
	let [firstName, , title] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];
	
	alert( title ); // Consul
	```
- In the code above, the second element of the array is skipped, the third one is assigned to `title`, and the rest of the array items are also skipped (as there are no variables for them).

---

#### Swap variables trick

- There’s a well-known trick for swapping values of two variables using a destructuring assignment:
	```javascript
	let guest = "Jane";
	let admin = "Pete";
	
	// Let's swap the values: make guest=Pete, admin=Jane
	[guest, admin] = [admin, guest];
	
	alert(`${guest} ${admin}`); // Pete Jane (successfully swapped!)
	```
- We can swap more than two variables this way.

---

#### Default values

- If the array is shorter than the list of variables on the left, there will be no errors. Absent values are considered undefined:
	```js
	let [firstName, surname] = [];
	
	alert(firstName); // undefined
	alert(surname); // undefined
	```
- If we want a “default” value to replace the missing one, we can provide it using `=`:
	```js
	// default values
	let [name = "Guest", surname = "Anonymous"] = ["Julius"];
	
	alert(name);    // Julius (from array)
	alert(surname); // Anonymous (default used)
	```
- Default values can be more **complex expressions or even function calls**. They are *evaluated only if the value is not provided*.
```js
// runs only prompt for surname
let [name = prompt('name?'), surname = prompt('surname?')] = ["Julius"];

alert(name);    // Julius (from array)
alert(surname); // whatever prompt gets
```
- Please note: the `prompt` will run only for the missing value (`surname`).