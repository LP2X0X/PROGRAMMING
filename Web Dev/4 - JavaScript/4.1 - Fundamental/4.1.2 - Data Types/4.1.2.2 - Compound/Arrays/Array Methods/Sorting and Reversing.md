---
tags:
  - js
  - array
  - methods
---

### 🔤 **`sort()`** – Sort alphabetically by default (The items are sorted as strings by default.)

```javascript
const names = ['John', 'Alice', 'Bob'];
names.sort();
console.log(names); // ['Alice', 'Bob', 'John']
```

---

### To use our own sorting order, we need to supply a function as the argument of `arr.sort()`.

- The function should compare two arbitrary values and return:
	```js
	function compare(a, b) {
	  if (a > b) return 1; // if the first value is greater than the second
	  if (a == b) return 0; // if values are equal
	  if (a < b) return -1; // if the first value is less than the second
	}
	```

- For instance, to sort as numbers:
	```js
	function compareNumeric(a, b) {
	  if (a > b) return 1;
	  if (a == b) return 0;
	  if (a < b) return -1;
	}
	
	let arr = [ 1, 2, 15 ];
	
	arr.sort(compareNumeric);
	
	alert(arr);  // 1, 2, 15
	```

- A comparison function is only required to return a positive number to say “greater” and a negative number to say “less”. That allows to write shorter functions:
	```javascript
	let arr = [ 1, 2, 15 ];
	
	arr.sort(function(a, b) { return a - b; });
	
	alert(arr);  // 1, 2, 15
	```

````ad-info
Use `localeCompare` for strings

Strings comparison algorithm compares letters by their codes by default.

For many alphabets, it’s better to use `str.localeCompare` method to correctly sort letters, such as `Ö`.

For example, let’s sort a few countries in German:
```javascript
let countries = ['Österreich', 'Andorra', 'Vietnam'];

alert( countries.sort( (a, b) => a > b ? 1 : -1) ); // Andorra, Vietnam, Österreich (wrong)

alert( countries.sort( (a, b) => a.localeCompare(b) ) ); // Andorra,Österreich,Vietnam (correct!)
```
````

---

### 🔁 **`reverse()`** – Reverse the order **in place**.

```javascript
const letters = ['a', 'b', 'c'];
letters.reverse();
console.log(letters); // ['c', 'b', 'a']
```
