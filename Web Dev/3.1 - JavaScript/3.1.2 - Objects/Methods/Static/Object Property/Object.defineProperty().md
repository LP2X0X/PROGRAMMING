---
tags: 
 - js
 - object
 - property
 - method
 - static
---

- The method [`Object.defineProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) allows you to **create or change** a property and control its behavior with flags (`writable`, `enumerable`, `configurable`).
    

---

### 🔹 Syntax:

- `obj`: The object where you want to define the property.
    
- `propertyName`: The name of the property.
    
- `descriptor`: An object with settings for the property.
    

```js
Object.defineProperty(obj, propertyName, descriptor);
```

---

### 🔹 Example:

```js
let user = {};

// Create a property "name" that is read-only
Object.defineProperty(user, "name", {
  value: "John",
  writable: false,     // cannot be changed
  enumerable: true,    // will show up in loops
  configurable: true   // can be deleted or redefined
});

console.log(user.name); // John

user.name = "Pete";     // ❌ ignored (or error in strict mode)
console.log(user.name); // John
```

```js
const account = { balance: 100 };

Object.defineProperty(account, "deposit", {
  value: function (amount) {
    this.balance += amount;
  },
  writable: true,
  enumerable: true
});

account.deposit(50);
console.log(account.balance); // 150

```

---

- To set all property descriptors at once, we can use the method [Object.defineProperties(obj, descriptors)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) (which is its plural form).
	- The syntax is:
		```js
		Object.defineProperties(obj, {
		  prop1: descriptor1,
		  prop2: descriptor2
		  // ...
		});
		```
	- For instance:
		```js
		Object.defineProperties(user, {
		  name: { value: "John", writable: false },
		  surname: { value: "Smith", writable: false },
		  // ...
		});
		```