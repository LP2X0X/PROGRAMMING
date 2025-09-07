---
tags: 
 - js
 - class
 - keyword
---

The `instanceof` operator allows to check whether an object belongs to a certain class. It also takes inheritance into account.

In another words, `instanceof` checks whether the **prototype** of a constructor appears anywhere in the prototype chain of an object.

```js
obj instanceof Constructor
```

Equivalent logic (simplified):

```js
let proto = Object.getPrototypeOf(obj);
while (proto) {
  if (proto === Constructor.prototype) return true;
  proto = Object.getPrototypeOf(proto);
}
return false;

// The above code is roughly translated to this
obj.__proto__ === Class.prototype?
obj.__proto__.__proto__ === Class.prototype?
obj.__proto__.__proto__.__proto__ === Class.prototype?
...
// if any answer is true, return true
// otherwise, if we reached the end of the chain, return false
```

---

## Examples

### Basic

```js
class A {}
class B extends A {}

const b = new B();

console.log(b instanceof B); // true
console.log(b instanceof A); // true
console.log(b instanceof Object); // true
```

Why?  
`b → B.prototype → A.prototype → Object.prototype → null`

---

### With Built-Ins

```js
const arr = [];
console.log(arr instanceof Array);  // true
console.log(arr instanceof Object); // true
```

Because:  
`arr → Array.prototype → Object.prototype`

---

### Extending Built-Ins

```js
class MyArray extends Array {}
const myArr = new MyArray();

console.log(myArr instanceof MyArray); // true
console.log(myArr instanceof Array);   // true
console.log(myArr instanceof Object);  // true
```

Chain:  
`myArr → MyArray.prototype → Array.prototype → Object.prototype → null`

---

## `Symbol.hasInstance`

`instanceof` can be customized with `Symbol.hasInstance`:

```js
class MyClass {
  static [Symbol.hasInstance](obj) {
    return Array.isArray(obj); // custom check
  }
}

console.log([] instanceof MyClass); // true
console.log({} instanceof MyClass); // false
```

```js
// setup instanceOf check that assumes that
// anything with canEat property is an animal
class Animal {
  static [Symbol.hasInstance](obj) {
    if (obj.canEat) return true;
  }
}

let obj = { canEat: true };

alert(obj instanceof Animal); // true: Animal[Symbol.hasInstance](obj) is called
```

So `instanceof` is not just about prototypes—you can override its behavior.

---

## Key Points

1. `instanceof` walks the prototype chain.
    
2. Works with classes, functions, and built-ins.
    
3. Can be customized via `Symbol.hasInstance`.
    
4. Subclassing built-ins (`Array`, `Error`, etc.) preserves correct `instanceof` checks.
    
