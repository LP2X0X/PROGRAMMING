---
tags:
  - js
  - term
  - fundamental
---

In JavaScript, an **iterable** is any object that implements the `@@iterator` **method** (i.e., has a method with the key `[Symbol.iterator]`), which returns an **iterator**.

Here’s how to implement a **custom iterable**:

--- 

### ✅ **Minimal Requirements to Be Iterable**

```js
const obj = {
  [Symbol.iterator]() {
    return {
      next() {
        return { value: ..., done: ... };
      }
    };
  }
};

```

|Syntax Part|Must-Have?|Purpose|
|---|---|---|
|`[Symbol.iterator]()`|✅ Yes|This is what defines the object as **iterable**.|
|`next()`|✅ Yes|Required on the iterator object to control the iteration process.|
|`{ value: ..., done: ... }`|✅ Yes|The `next()` method must return this object to indicate state.|
|`value` (inside returned object)|✅ Yes|The actual value for the current iteration step.|
|`done` (inside returned object)|✅ Yes|A boolean telling whether iteration is complete.|

---

### ✅ Common Iterable Types:

- **Array**
- **[[Web Dev/4 - JavaScript/4.1 - Fundamental/Data Types/Primitives/Strings/String|String]]**
- **Map**
- **Set**
- **TypedArray** (e.g. `Uint8Array`)
- **arguments** object
- **DOM collections** (like `NodeList`)
- **Custom objects with `[Symbol.iterator]()`**

> ❌ `Object` (plain `{}`) is **not** iterable by default.

---

### ✅ Basic Custom Iterable Example

```js
let range = {
  from: 1,
  to: 5
};

// 1. call to for..of initially calls this
range[Symbol.iterator] = function() {

  // ...it returns the iterator object:
  // 2. Onward, for..of works only with the iterator object below, asking it for next values
  return {
    current: this.from,
    last: this.to,

    // 3. next() is called on each iteration by the for..of loop
    next() {
      // 4. it should return the value as an object {done:.., value :...}
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

// now it works!
for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```

```js
const myIterable = {
  data: [1, 2, 3],
  
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for (const item of myIterable) {
  console.log(item); // Output: 1, 2, 3
}
```

---

### ✅ Explanation

- `Symbol.iterator` is a special key used by the `for...of` loop.
    
- The function returns an **iterator**: an object with a `next()` method.
    
- The `next()` method returns an object with `{ value, done }`.
    

---

### ✅ Use with Spread Operator or Destructuring

Once it's iterable, you can use it like this:

```js
console.log([...myIterable]);     // [1, 2, 3]
const [a, b] = myIterable;        // a = 1, b = 2
```

---

### ✅ Generator-Based Iterable (Simpler)

Using a **generator function** makes iterables cleaner:

```js
const myIterable = {
  data: [1, 2, 3],
  *[Symbol.iterator]() {
    for (let item of this.data) {
      yield item;
    }
  }
};

for (let val of myIterable) {
  console.log(val); // 1, 2, 3
}
```

---

### Array.from

- There’s a universal method Array.from that takes an iterable or [[Array-like Objects|array-like]] value and makes a “real” `Array` from it. Then we can call array methods on it.
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

