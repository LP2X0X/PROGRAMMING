In **JavaScript**, every object is linked to a prototype from which it can inherit properties and methods. This mechanism is known as **prototypal inheritance**.

---

### 📌 **Object Prototype Basics**

1. Every object in JavaScript has an internal property called `[[Prototype]]` which points to another object (its **prototype**).
2. You can access this prototype using:
    - **`Object.getPrototypeOf(obj)`** (recommended)
    - **`__proto__`** (deprecated but still works)
3. **`Object.prototype`** is the top-level prototype, from which all other objects inherit.

---

### ✅ **Example of Object Prototype**

```javascript
const obj = {}; // Creates an empty object
console.log(Object.getPrototypeOf(obj)); // Outputs: [Object: null prototype] {}
```

Here, the object `obj` inherits from `Object.prototype`.

---

### 🛠️ **Common Properties and Methods on `Object.prototype`**

All objects inherit these by default unless the prototype chain is broken:

|**Property/Method**|**Description**|
|---|---|
|`constructor`|Refers to the function that created the object.|
|`hasOwnProperty()`|Checks if the object itself has a property (not inherited).|
|`isPrototypeOf()`|Checks if an object exists in another object's prototype chain.|
|`propertyIsEnumerable()`|Checks if a property is enumerable.|
|`toString()`|Returns a string representation of the object.|
|`valueOf()`|Returns the primitive value of an object.|

---

### 🔍 **Examples of Each Property/Method**

1. **`constructor`**

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person('John');
console.log(john.constructor === Person); // true
```

2. **`hasOwnProperty()`**

```javascript
const car = { brand: 'Toyota' };
console.log(car.hasOwnProperty('brand')); // true
console.log(car.hasOwnProperty('toString')); // false (inherited)
```

3. **`isPrototypeOf()`**

```javascript
const animal = { type: 'mammal' };
const dog = Object.create(animal);
console.log(animal.isPrototypeOf(dog)); // true
```

4. **`toString()`**

```javascript
const obj = { a: 1 };
console.log(obj.toString()); // [object Object]
```

5. **`valueOf()`**

```javascript
const num = new Number(42);
console.log(num.valueOf()); // 42
```

---

### 📚 **Prototype Chain in Action**

If you try to access a property on an object:

1. **JavaScript** checks the object itself.
2. If not found, it moves up the **prototype chain**.
3. This continues until it finds the property or reaches `null`.

```javascript
const person = {
  greet() {
    console.log('Hello!');
  },
};

const student = Object.create(person);
student.study = function() {
  console.log('Studying...');
};

student.greet(); // Inherited from person: "Hello!"
student.study(); // Own method: "Studying..."
```

---

### 🔥 **Customizing the Prototype**

You can directly modify or extend `Object.prototype`—but do it carefully as it affects **all** objects:

```javascript
Object.prototype.sayHi = function() {
  console.log('Hi from prototype!');
};

const obj = {};
obj.sayHi(); // Outputs: Hi from prototype!
```

Would you like more examples or details on extending or manipulating prototypes?