---
tags:
 - js
 - object
 - method
 - fundamental
---

## 🔹 Static Methods (`Object.*`)

### ✅ Creation & Cloning

- **`Object.create(proto, properties?)`** → creates a new object with the given prototype.
    
    ```js
    const obj = Object.create(null); // no prototype
    ```
    
- **`Object.assign(target, ...sources)`** → shallow copies properties into target.
    
    ```js
    const copy = Object.assign({}, { a: 1 }, { b: 2 });
    // { a: 1, b: 2 }
    ```
    
- **`structuredClone(obj)`** → deep copy (supports most built-in types).
    

---

### ✅ Keys, Values, Entries

- **`Object.keys(obj)`** → array of keys.
    
- **`Object.values(obj)`** → array of values.
    
- **`Object.entries(obj)`** → array of `[key, value]` pairs.
    
- **`Object.fromEntries(entries)`** → builds object from `[key, value]` pairs.
    
    ```js
    const obj = Object.fromEntries([["a", 1], ["b", 2]]);
    // { a: 1, b: 2 }
    ```
    

---

### ✅ Property Descriptors

- **`Object.getOwnPropertyDescriptor(obj, prop)`** → descriptor for a property.
    
- **`Object.getOwnPropertyDescriptors(obj)`** → descriptors for all props.
    
- **`Object.defineProperty(obj, prop, descriptor)`** → define or change a property with control over `enumerable`, `writable`, `configurable`.
    
- **`Object.defineProperties(obj, descriptors)`** → define multiple properties at once.
    

---

### ✅ Object Information

- **`Object.getOwnPropertyNames(obj)`** → array of own string keys.
    
- **`Object.getOwnPropertySymbols(obj)`** → array of own `Symbol` keys.
    
- **`Object.getOwnPropertyDescriptors(obj)`** → all descriptors.
    
- **`Object.getOwnPropertyNames(obj)`** → string keys only.
    
- **`Reflect.ownKeys(obj)`** → all keys (string + symbol).
    

---

### ✅ Prototypes

- **`Object.getPrototypeOf(obj)`** → get prototype.
    
- **`Object.setPrototypeOf(obj, proto)`** → set prototype.
    

---

### ✅ Freezing & Sealing

- **`Object.freeze(obj)`** → makes object immutable (can’t change or add properties).
    
- **`Object.isFrozen(obj)`** → check if frozen.
    
- **`Object.seal(obj)`** → prevent adding/removing props (but values can still change).
    
- **`Object.isSealed(obj)`** → check if sealed.
    
- **`Object.preventExtensions(obj)`** → prevent adding new props.
    
- **`Object.isExtensible(obj)`** → check if extensible.
    

---

### ✅ Equality & Safety

- **`Object.is(value1, value2)`** → like `===` but better handles `NaN` and `-0`.
    
    ```js
    Object.is(NaN, NaN); // true
    Object.is(-0, 0);    // false
    ```
    
- **`Object.hasOwn(obj, key)`** (ES2022) → check if object has own property (no prototype lookup).
    

---

## 🔹 Prototype Methods (on all objects)

These are inherited from `Object.prototype`, so available on any object:

- **`obj.hasOwnProperty(key)`** → check if key exists directly on object.
    
- **`obj.isPrototypeOf(obj2)`** → check if object is in prototype chain.
    
- **`obj.propertyIsEnumerable(key)`** → check if property is enumerable.
    
- **`obj.toString()`** → returns object type as string (`[object Object]`).
    
- **`obj.valueOf()`** → returns primitive value (default: object itself).
    

---

## 🔹 Quick Examples

```js
const user = { name: "Alice", age: 25 };

// Keys, values, entries
Object.keys(user);     // ["name", "age"]
Object.values(user);   // ["Alice", 25]
Object.entries(user);  // [["name","Alice"],["age",25]]

// Build object from entries
Object.fromEntries([["a", 1], ["b", 2]]); // { a: 1, b: 2 }

// Cloning
const clone = Object.assign({}, user);

// Descriptors
Object.getOwnPropertyDescriptor(user, "name");
// { value: "Alice", writable: true, enumerable: true, configurable: true }

// Freeze & check
Object.freeze(user);
console.log(Object.isFrozen(user)); // true

// Equality
Object.is(NaN, NaN); // true
Object.is(-0, 0);    // false
```

---

## 🔹 Summary Table

|Category|Methods|
|---|---|
|**Creation**|`create`, `assign`, `structuredClone`|
|**Keys/Values**|`keys`, `values`, `entries`, `fromEntries`|
|**Descriptors**|`getOwnPropertyDescriptor(s)`, `defineProperty(ies)`|
|**Prototype**|`getPrototypeOf`, `setPrototypeOf`|
|**Sealing**|`freeze`, `isFrozen`, `seal`, `isSealed`, `preventExtensions`, `isExtensible`|
|**Equality**|`is`, `hasOwn`|
|**Prototype methods**|`hasOwnProperty`, `isPrototypeOf`, `propertyIsEnumerable`, `toString`, `valueOf`|