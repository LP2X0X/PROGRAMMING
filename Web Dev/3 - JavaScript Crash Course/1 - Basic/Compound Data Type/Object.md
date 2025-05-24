---
tags: js, datatype, fundamental
---
### **🧱 JavaScript Objects — The Basics**


In JavaScript, an **object** is a collection of **key-value pairs** used to store structured data.

---

### **✅ Creating an Object**
```ad-important
JS object keys are always strings or symbols. Even if you write obj[42] = "hello", the key is "42".
```

- Create object using [[Object Literal|object literal]].

```js
const person = {
  name: "Alice",
  age: 30,
  isStudent: false
};
```

```ad-note
All object keys are strings, but if your key is a [[Valid Identifier|valid identifier]], it’s common practice to omit the quotes.
```

---

### **🛠 Accessing Properties**

- **Dot notation**:
    - For keys that are [[Valid Identifier|valid identifiers]], you can use dot notation instead of square brackets, with the key name coming after the dot.

```js
console.log(person.name); // "Alice"
```

- **Bracket notation**:
    

```js
console.log(person["age"]); // 30
```

- Use bracket notation when the key is dynamic or contains spaces:
    

```js
const key = "isStudent";
console.log(person[key]); // false
```

  

---

### **✍️ Modifying Properties**

```js
person.age = 31;
person["name"] = "Bob";
```

---

### **➕ Adding Properties**

```js
person.country = "USA";
```

---

### **❌ Deleting Properties**

```js
delete person.isStudent;
```

---

### **🔁 Looping Through Properties**

```js
for (let key in person) {
  console.log(key, person[key]);
}
```

---

### **📦 Useful Object Methods**
- These are all static methods of Object type:

|**Method**|**Description**|
|---|---|
|Object.keys(obj)|Returns array of keys.|
|Object.values(obj)|Returns array of values.|
|Object.entries(obj)|Returns array of [key, value] pairs.|
|Object.assign(target, source)|Copies properties from one to another.|
|Object.hasOwnProperty(key)|Checks if key exists in object.|
|Object.freeze(obj)|Makes object immutable.|

---

### **🧬 Nested Objects**

```
const user = {
  name: "Jane",
  address: {
    city: "Paris",
    zip: 75000
  }
};

console.log(user.address.city); // "Paris"
```

---

### **🆕 Destructuring (ES6)**

```
const { name, age } = person;
```