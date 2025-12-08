---
tags: 
 - js
 - array
 - method
---

`Array.from()` is a **built-in static method** that creates a new array from:

1. **[[Array-like Objects|Array-like objects]]** (e.g. `arguments`, `NodeList`, `Set`, `Map.keys()`)
    
2. **[[Iterables|Iterable objects]]** (e.g. strings, generator results, etc.)
    

- Here's an example:

	```javascript
	let arrayLike = {
	  0: "Hello",
	  1: "World",
	  length: 2
	};
	
	let arr = Array.from(arrayLike); // (*)
	alert(arr.pop()); // World (method works)
	```

- The full syntax for `Array.from` also allows us to provide an optional “mapping” function:
	```javascript
	Array.from(obj[, mapFn, thisArg])
	```

- The optional second argument `mapFn` can be a function that will be applied to each element before adding it to the array, and [[thisArg]] allows us to set `this` for it.
	```js
	let range = {
	  from: 1,
	  to: 5
	};
	
	// square each number
	let arr = Array.from(range, num => num * num);
	
	alert(arr); // 1,4,9,16,25
	```

- This is a great way to create array with a specific number of items:
	```js
	const arr = Array.from({ length: 5 }); 
	console.log(arr); // [ undefined, undefined, undefined, undefined, undefined ]
	
	const arr = Array.from({ length: 5 }, () => 0); 
	console.log(arr); // [ 0, 0, 0, 0, 0]
	```

---

### 🧠 Basic Syntax

```js
Array.from(arrayLike [, mapFn [, thisArg]])
```

#### Parameters

|Parameter|Type|Description|
|---|---|---|
|`arrayLike`|Array-like or iterable|The source to convert into an array|
|`mapFn`|Function _(optional)_|A mapping function applied to each element before insertion|
|`thisArg`|Any _(optional)_|Value to use as `this` inside `mapFn`|

---

## 🧩 Common Examples

### 1️⃣ Convert an array-like object

```js
function demo() {
  console.log(arguments);             // [Arguments] { '0': 1, '1': 2, '2': 3 }
  const arr = Array.from(arguments);
  console.log(arr);                   // [1, 2, 3]
}
demo(1, 2, 3);
```

✅ Converts `arguments` (which isn’t a real array) into a true array.

---

### 2️⃣ Convert a string to an array of characters

```js
Array.from('hello');   // ['h', 'e', 'l', 'l', 'o']
```

---

### 3️⃣ Create an array with a mapping function

```js
Array.from({ length: 5 }, (_, i) => i * 2);
// [0, 2, 4, 6, 8]
```

Here:

- `{ length: 5 }` is an **array-like object** (no elements, but a length)
    
- `(_, i) => i * 2` is the mapping function
    
- `_` means “ignore the value” (because there isn’t one)
    

---

### 4️⃣ Convert `Set` or `Map` to array

```js
const s = new Set([1, 2, 3]);
Array.from(s);               // [1, 2, 3]

const m = new Map([['a', 1], ['b', 2]]);
Array.from(m);               // [['a', 1], ['b', 2]]
Array.from(m.values());      // [1, 2]
Array.from(m.keys());        // ['a', 'b']
```

---

### 5️⃣ Fill arrays dynamically

```js
const arr = Array.from({ length: 10 }, (_, i) => i + 1);
console.log(arr); // [1,2,3,4,5,6,7,8,9,10]
```

✅ Very common pattern for generating numeric sequences.

---

## 🧩 Difference vs Other Methods

|Method|Description|When to Use|
|---|---|---|
|`Array.from()`|Converts or builds arrays from any iterable|Best for flexible creation / transformation|
|`Array.of()`|Creates array from given arguments|`Array.of(1,2,3)` → `[1,2,3]`|
|`[...iterable]`|Spread syntax to unpack elements|Simpler alternative, but lacks mapping function|

Example:

```js
[...'abc'];           // ['a','b','c']
Array.from('abc');    // ['a','b','c'] → same
Array.from('abc', c => c.toUpperCase()); // ['A','B','C'] → only possible with mapFn
```

---

## ⚙️ Summary

|Feature|Description|
|---|---|
|Converts any iterable / array-like to array|✅|
|Accepts optional mapping function|✅|
|Provides optional `this` binding for mapFn|✅|
|Commonly used for range generation & conversions|✅|
|Alternative to `[...iterable]`|✅|

---

### 🧩 Quick Cheatsheet

```js
// Convert iterable
Array.from('abc')               // ['a','b','c']

// Convert Set or Map
Array.from(new Set([1,2,3]))    // [1,2,3]

// Create array of N elements
Array.from({ length: 5 }, (_, i) => i)  // [0,1,2,3,4]

// Clone array
Array.from([1,2,3])             // [1,2,3]
```