Here’s a clear comparison between null and undefined in JavaScript:

---

### **🔍 What They Are**

|**Feature**|undefined|null|
|---|---|---|
|**Meaning**|A variable has been declared but not assigned a value.|A variable has been explicitly assigned “no value”.|
|**Set By**|JavaScript (automatically)|Developer (manually)|
|**Type**|undefined (type: "undefined")|object (yes, this is a legacy bug in JS)|
|**Use Case**|Default value for uninitialized variables or missing function returns.|Intentional placeholder for “no value” or “empty”.|
|**Truthy/Falsy**|Falsy|Falsy|

---

### **🧪 Examples**

#### **undefined**

```js
let x;
console.log(x); // undefined

function foo() {}
console.log(foo()); // undefined (no return value)

let obj = {};
console.log(obj.key); // undefined (key doesn't exist)
```

#### **null**

```js
let y = null;
console.log(y); // null (set intentionally)

let user = {
  name: null // we know there is no name, not just missing
};
```

---

### **🔁 Equality Comparison**

```js
undefined == null   // true (they are loosely equal)
undefined === null  // false (different types)
```

---

### **✅ When to Use**

- Use undefined when a variable hasn’t been assigned yet or is missing.
    
- Use null when you **want to explicitly say**: “there is no value here.”
    
