---
tags: 
 - js
 - object
 - method
---

## **📖 hasOwnProperty**

obj.hasOwnProperty(prop) → returns **true** if the property exists **directly on the object itself**, not on its prototype chain.

---

### **🔍 Example**

```js
const animal = { eats: true };
const rabbit = Object.create(animal);

rabbit.jumps = true;

console.log(rabbit.jumps);              // true  (own property)
console.log(rabbit.eats);               // true  (in prototype)
console.log(rabbit.hasOwnProperty("jumps")); // true
console.log(rabbit.hasOwnProperty("eats"));  // false
```

- jumps → belongs **to rabbit itself**.
    
- eats → comes **from prototype (animal)**, so hasOwnProperty is false.
    

---

### **🛠️ Why use it?**

Because a normal property check (in or obj.prop !== undefined) **includes prototype chain**:

```js
console.log("eats" in rabbit); // true (because prototype has it)
console.log("jumps" in rabbit); // true
```

If you need to **distinguish own properties from inherited ones**, always use hasOwnProperty.

---

### **⚠️ Edge Case**

If someone overwrites hasOwnProperty on the object, you should use it **safely via Object.prototype**:

```js
Object.prototype.hasOwnProperty.call(rabbit, "eats"); // false
```

---

✅ **In short:**

- obj.hasOwnProperty(key) → checks only own keys.
    
- "key" in obj → checks own keys + prototype chain.
    