---
tags: js, keyword, fundamental
---

### **🔁 for...of in JavaScript**

- The for...of loop lets you **iterate over iterable objects** like:
	- Arrays
	- Strings
	- Maps
	- Sets
	- DOM collections
	- Arguments object

---

### **✅ Basic Syntax**

```js
for (const element of iterable) {
  // use element
}

for (let element of iterable) {
  // use element
}
```

---

### **📦 Examples**

#### **1. Array**

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
// Output: apple, banana, cherry
```

#### **2. String**

```js
for (const char of "hello") {
  console.log(char);
}
// Output: h, e, l, l, o
```

#### **3. Set**

```js
const letters = new Set(["a", "b", "c"]);

for (const letter of letters) {
  console.log(letter);
}
```

#### **4. Map**

```js
const user = new Map([
  ["name", "Alice"],
  ["age", 30]
]);

for (const [key, value] of user) {
  console.log(`${key}: ${value}`);
}
```

---

### **❌ Not for plain objects ( {} )**

```js
const obj = { a: 1, b: 2 };

for (const item of obj) {
  // ❌ TypeError: obj is not iterable
}
```

For objects, use [[for in|for...in]] or Object.keys():

```js
for (const key in obj) {
  console.log(`${key}: ${obj[key]}`);
}
```

Or:

```js
for (const key of Object.keys(obj)) {
  console.log(`${key}: ${obj[key]}`);
}
```

---

### **✅ Summary**

|**Loop Type**|**Use With**|
|---|---|
|for...of|Arrays, strings, maps, sets|
|for...in|Object **keys**|
|.forEach()|Arrays (function-based)|
