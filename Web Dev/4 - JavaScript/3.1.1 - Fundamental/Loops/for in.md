---
tags:
  - js
  - keyword
  - fundamental
---

The for...in loop is used to **iterate over the [[Property Flags#3. \*\*\`enumerable\`\*\*|enumerable]] property keys** of an [[Web Dev/4 - JavaScript/3.1.2 - Objects/Terms/Cores/Object|object]].

---

### **✅ Syntax**

```js
for (let key in object) {
  // use key and object[key]
}
```

---

### **📦 Example with Object**

```js
const person = {
  name: "Alice",
  age: 25,
  city: "Paris"
};

for (let key in person) {
  console.log(`${key}: ${person[key]}`);
}
// Output:
// name: Alice
// age: 25
// city: Paris
```

---

### **⚠️ Behavior with Arrays**

```js
const fruits = ["apple", "banana", "cherry"];

for (let index in fruits) {
  console.log(index, fruits[index]);
}
// Output:
// 0 apple
// 1 banana
// 2 cherry
```

> ✅ It works, but not recommended for arrays. Use for...of or forEach() instead.

---

### **🧠 Use for...in when:**

- Iterating **over object keys**
    
- You want both the **key** and the **value**
    

```ad-note
The `for..in` loop iterates over inherited properties too.
```

---

### **❌ Avoid for...in with:**

- Arrays (because it iterates over **keys**, not values)
- Objects with inherited properties (unless filtered)

Use this check to **avoid inherited properties**:

```js
for (let key in obj) {
  if (obj.hasOwnProperty(key)) {
    // safe to use key
  }
}
```

---

### **✅ Summary**

|**Loop**|**Best For**|
|---|---|
|for...in|Object keys|
|for...of|Array elements, strings|
|forEach()|Arrays (functional)|