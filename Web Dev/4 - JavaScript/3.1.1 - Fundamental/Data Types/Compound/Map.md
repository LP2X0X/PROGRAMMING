---
tags:
  - js
  - datatype
  - fundamental
---

- [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) is a collection of keyed data items, just like an `Object`. But the main difference is that `Map` allows keys of any type.
- Methods and properties are:
	- [`new Map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Map) – creates the map.
	- [`map.set(key, value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/set) – stores the value by the key.
	- [`map.get(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get) – returns the value by the key, `undefined` if `key` doesn’t exist in map.
	- [`map.has(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/has) – returns `true` if the `key` exists, `false` otherwise.
	- [`map.delete(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/delete) – removes the element (the key/value pair) by the key.
	- [`map.clear()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/clear) – removes everything from the map.
	- [`map.size`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/size) – returns the current element count.
- For instance:
	```js
	let map = new Map();
	
	map.set('1', 'str1');   // a string key
	map.set(1, 'num1');     // a numeric key
	map.set(true, 'bool1'); // a boolean key
	
	// remember the regular Object? it would convert keys to string
	// Map keeps the type, so these two are different:
	alert( map.get(1)   ); // 'num1'
	alert( map.get('1') ); // 'str1'
	
	alert( map.size ); // 3
	```

	```js
	const map = new Map([
	  ["key1", "value1"],
	  ["key2", "value2"],
	]);
	
	console.log(map.get("key1")); // "value1"
	```

----

### Gotcha: How `Map` compares keys

- To test keys for equivalence, `Map` uses the algorithm [SameValueZero](https://tc39.github.io/ecma262/#sec-samevaluezero). **It is roughly the same as strict equality `===`**, but the difference is that `NaN` is considered equal to `NaN`. So `NaN` can be used as the key as well.

```ad-important
When you store **non-primitive values** (objects, arrays, functions) as `Map` keys, JavaScript compares them **by reference**, not by content.
```

- This algorithm can’t be changed or customized.

----

### Map can also use objects as keys
- Using objects as keys is one of the most notable and important `Map` features. The same does not count for `Object`. String as a key in `Object` is fine, but we can’t use another `Object` as a key in `Object`.
	```js
	let john = { name: "John" };
	
	// for every user, let's store their visits count
	let visitsCountMap = new Map();
	
	// john is the key for the map
	visitsCountMap.set(john, 123);
	
	alert( visitsCountMap.get(john) ); // 123
	```

----

### Chaining

- Every `map.set` call returns the map itself, so we can “chain” the calls:
	```js
	map.set('1', 'str1')
	  .set(1, 'num1')
	  .set(true, 'bool1');
	```

----

### Iteration over Map

- For looping over a `map`, there are 3 methods:
	- [`map.keys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/keys) – returns an iterable for keys,
	- [`map.values()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/values) – returns an iterable for values,
	- [`map.entries()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/entries) – returns an iterable for entries `[key, value]`, it’s used by default in `for..of`.
- Examples:
	```js
	let recipeMap = new Map([
	  ['cucumber', 500],
	  ['tomatoes', 350],
	  ['onion',    50]
	]);
	
	// iterate over keys (vegetables)
	for (let vegetable of recipeMap.keys()) {
	  alert(vegetable); // cucumber, tomatoes, onion
	}
	
	// iterate over values (amounts)
	for (let amount of recipeMap.values()) {
	  alert(amount); // 500, 350, 50
	}
	
	// iterate over [key, value] entries
	for (let entry of recipeMap) { // the same as of recipeMap.entries()
	  alert(entry); // cucumber,500 (and so on)
	}
	```
- Map also has a built-in `forEach` method, similar to `Array`:
	```js
	// runs the function for each (key, value) pair
	recipeMap.forEach( (value, key, map) => {
	  alert(`${key}: ${value}`); // cucumber: 500 etc
	});
	```

---

### Object.entries - Map from Object

- If we have a plain object, and we’d like to create a `Map` from it, then we can use built-in method [Object.entries(obj)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries) that returns an array of key/value pairs for an object exactly in that format.

- So we can create a map from an object like this:

```js
let obj = {
  name: "John",
  age: 30
};

let map = new Map(Object.entries(obj));

alert( map.get('name') ); // John
```

- Here, `Object.entries` returns the array of key/value pairs: `[ ["name","John"], ["age", 30] ]`. That’s what `Map` needs.

---

### Object.fromEntries: Object from Map

- There’s `Object.fromEntries` method that does the reverse: given an array of `[key, value]` pairs, it creates an object from them:
	```js
	let prices = Object.fromEntries([
	  ['banana', 1],
	  ['orange', 2],
	  ['meat', 4]
	]);
	
	// now prices = { banana: 1, orange: 2, meat: 4 }
	
	alert(prices.orange); // 2
	```

	```js
	let map = new Map();
	map.set('banana', 1);
	map.set('orange', 2);
	map.set('meat', 4);
	
	let obj = Object.fromEntries(map.entries()); // make a plain object (*)
	
	// done!
	// obj = { banana: 1, orange: 2, meat: 4 }
	
	alert(obj.orange); // 2
	```

A call to `map.entries()` returns an iterable of key/value pairs, exactly in the right format for `Object.fromEntries`.

We could also make line `(*)` shorter:

```javascript
let obj = Object.fromEntries(map); // omit .entries()
```

That’s the same, because `Object.fromEntries` expects an iterable object as the argument. Not necessarily an array. And the standard iteration for `map` returns same key/value pairs as `map.entries()`. So we get a plain object with same key/values as the `map`.