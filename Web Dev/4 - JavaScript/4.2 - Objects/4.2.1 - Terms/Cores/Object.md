---
tags:
  - js
  - datatype
  - fundamental
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

```ad-warning
A notable feature of objects in JavaScript, compared to many other languages, is that it’s possible to access any property. There will be no error if the property doesn’t exist!
```

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

 - Access a **non-existent property** of an object return undefined.
    

```js
const obj = { a: 1 };

console.log(obj.a);   // 1
console.log(obj.b);   // undefined (since "b" does not exist)
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

- Or we can use [[Computed Property|computed property]]:

```js
let fruit = "Apple";

let bag = {
	[fruit]: 1,
};
```

- Also, here's **object property shorthand**:

```js
const name = "Alice";
const age = 25;

const person = {
  name,
  age,
};

console.log(person);
// Output: { name: "Alice", age: 25 }
```

---

### **❌ Deleting Properties**

```js
delete person.isStudent;
```

- `delete obj.prop` removes the property from the object.
- Returns `true` if successful.
- Does **not** affect prototype properties.
- Slower than setting `prop = undefined`, but actually **removes the key**.

---

### **🔁 Looping Through Properties**

```js
for (let key in person) {
  console.log(key, person[key]);
}
```

---


### **🧬 Nested Objects**

```js
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