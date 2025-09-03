---
tags:
  - js
  - array
  - methods
---

### 🔘 **`every()`** – Check if **all** elements match a condition.

```javascript
const ages = [25, 30, 35];
console.log(ages.every(age => age >= 18)); // true
```

### 🔴 **`some()`** – Check if **at least one** matches.

```javascript
const scores = [45, 60, 80];
console.log(scores.some(score => score > 75)); // true
```

### **Array.isArray()**

- Arrays do not form a separate language type. They are based on objects.
- So `typeof` does not help to distinguish a plain object from an array:
	```javascript
	alert(typeof {}); // object
	alert(typeof []); // object (same)
	```

- But arrays are used so often that there’s a special method for that: Array.isArray(value). It returns `true` if the `value` is an array, and `false` otherwise.
	```javascript
	alert(Array.isArray({})); // false
	
	alert(Array.isArray([])); // true
	```