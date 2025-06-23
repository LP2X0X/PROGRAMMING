---
tags: js, method, fundamental
---

## **🧾 Object.assign() - What it does **

Object.assign(target, ...sources) **copies properties** from one or more **source objects** into a **target object**, and returns the target.

---

### **✅ Basic Usage**

```js
const target = { a: 1 };
const source = { b: 2, c: 3 };

Object.assign(target, source);
// target is now: { a: 1, b: 2, c: 3 }
```

- It **modifies and returns target**

---

## **🔁 Multiple Sources**

```js
Object.assign({}, obj1, obj2, obj3);
```

- Later sources **override** earlier ones:

```js
Object.assign({ a: 1 }, { a: 2, b: 3 }); // → { a: 2, b: 3 }
```

---

## **⚠️ It does not deep copy nested object **

```js
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

alert( user.sizes === clone.sizes ); // true, same object

// user and clone share sizes
user.sizes.width = 60;    // change a property from one place
alert(clone.sizes.width); // 60, get the result from the other one
```

---

## **🧠 Summary**

- 🔧 **Copies own enumerable properties** from source(s) to target
    
- 📦 **Returns the modified target object**
    
- ⚠️ Doesn’t copy symbol or non-enumerable props
    
