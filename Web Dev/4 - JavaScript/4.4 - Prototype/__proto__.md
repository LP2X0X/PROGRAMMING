---
tags: 
 - js
 - object
 - property
 - advance
---

In **JavaScript**, every object is linked to a prototype from which it can inherit properties and methods. This mechanism is known as **prototypal inheritance**.
![[Pasted image 20250826145342.png|center|200]] ^e54aa0

```ad-note
The only usage of `__proto__`, that’s not frowned upon, is as a property when creating a new object: `{ __proto__: ... }`.
```

---

### 📌 **Object Prototype Basics**

1. Every object in JavaScript has an internal property called `[[Prototype]]` which points to another object (its **prototype**).
2. You can access this prototype using:
    - **`Object.getPrototypeOf(obj)`** (recommended)
    - `__proto__` (deprecated but still works) is a way to access `[[Prototype]]`, it is not `[[Prototype]]` itself.
		```js
		let rabbit = {
		  jumps: true,
		  __proto__: animal
		};
		```
3. **`Object.prototype`** is the top-level prototype, from which all other objects inherit.

---

### ✅ **Example of Object Prototype**

```js
let animal = {
  eats: true,
  walk() {
    alert("Animal walk");
  }
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

let longEar = {
  earLength: 10,
  __proto__: rabbit
};

// walk is taken from the prototype chain
longEar.walk(); // Animal walk
alert(longEar.jumps); // true (from rabbit)
```

![[Pasted image 20250906111051.png|center|300]]

- Now if we read something from `longEar`, and it’s missing, JavaScript will look for it in `rabbit`, and then in `animal`.

```ad-note
This is an indication that when an object is inherited from another object using \_\_proto\_\_, it gets all the parent's property.
```

^57e3c2

---

### **🔍 Writting does not use prototype**

1. **Reading (lookup) → checks prototype chain**
```js
const animal = { eats: true };
const rabbit = Object.create(animal);

console.log(rabbit.eats); // true  <-- found in prototype (animal)
```
- JS didn’t find eats directly on rabbit.
- It looked in rabbit.__proto__ (animal), found it, and returned it.

2. **Writing (assignment and only assignment, not modifying) → always goes to the object itself**
```js
rabbit.eats = false; 
console.log(rabbit.eats); // false (own property now)
console.log(animal.eats); // true  (unchanged)
```

```js
let animal = {
  eats: true,
  walk() {
	/* this method won't be used by rabbit */
  }
};

let rabbit = {
  __proto__: animal
};

rabbit.walk = function() {
  alert("Rabbit! Bounce-bounce!");
};

rabbit.walk(); // Rabbit! Bounce-bounce!
```

- Even though eats exists in the prototype (animal),
- assigning creates/updates a **new property on rabbit**, not the prototype.



3. **Deleting**
```js
delete rabbit.eats;
console.log(rabbit.eats); // true (back to reading from prototype)
```

#### **🛠️ Summary**

- ✅ **Read** → JS looks at the object, and if not found, continues up the prototype chain.
    
- ✅ **Write** → JS never writes to the prototype; it creates/updates the property on the object itself.
    
- ✅ **Delete** → removes only from the object’s own properties, not from the prototype.
    

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

---

### ❓The value of `this`

An interesting question may arise in the example above: what’s the value of `this` inside `set fullName(value)`? Where are the properties `this.name` and `this.surname` written: into `user` or `admin`?

The answer is simple: `this` is not affected by prototypes at all.

**No matter where the method is found: in an object or its prototype. In a method call, `this` is always the object before the dot.**

---

### **📌 Objects and their prototype**

```js
const animal = { eats: true };
const rabbit = Object.create(animal);

console.log(rabbit.eats); // true
```

If you **modify** the prototype object (animal):

```js
animal.walks = true;
console.log(rabbit.walks); // ✅ true
```

But if you **reassign** the prototype variable:

```Js
const newAnimal = { sleeps: true };
rabbit.__proto__ = animal; // was set at creation

// Changing "animal" variable reference does nothing
animal = newAnimal; // ❌ doesn't affect rabbit
console.log(rabbit.sleeps); // undefined
```

Because rabbit.[[__proto__]] was already pointing to the old object (animal), not to the variable.
