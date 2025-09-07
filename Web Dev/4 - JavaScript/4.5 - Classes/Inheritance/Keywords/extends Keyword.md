---
tags:
  - js
  - class
  - inheritance
  - keyword
---

- Let’s say we have class `Animal`:
	```js
	class Animal {
	  constructor(name) {
	    this.speed = 0;
	    this.name = name;
	  }
	  run(speed) {
	    this.speed = speed;
	    alert(`${this.name} runs with speed ${this.speed}.`);
	  }
	  stop() {
	    this.speed = 0;
	    alert(`${this.name} stands still.`);
	  }
	}
	
	let animal = new Animal("My animal");
	```
- Here’s how we can represent `animal` object and `Animal` class graphically:
![[Pasted image 20250903160310.png|center|400]]
- As rabbits are animals, `Rabbit` class should be based on `Animal`, have access to animal methods, so that rabbits can do what “generic” animals can do.
```ad-note
The syntax to extend another class is: `class Child extends Parent`.
```

</br>

- Let’s create `class Rabbit` that inherits from `Animal`:
	```js
	class Rabbit extends Animal {
	  hide() {
	    alert(`${this.name} hides!`);
	  }
	}
	
	let rabbit = new Rabbit("White Rabbit");
	
	rabbit.run(5); // White Rabbit runs with speed 5.
	rabbit.hide(); // White Rabbit hides!
	```

</br>

- Internally, `extends` keyword works using the good old prototype mechanics. It sets `Rabbit.prototype.[[Prototype]]` to `Animal.prototype`. So, if a method is not found in `Rabbit.prototype`, JavaScript takes it from `Animal.prototype`.
![[Pasted image 20250904074913.png|center|400]]

- Remember that prototype is an object, therefore it also has the \_\_proto\_\_ property. Also, from [[__proto__#^57e3c2|this note]], we can understand how the use of the above method work.


````ad-note
Any expression is allowed after `extends`

Class syntax allows to specify not just a class, but any expression after `extends`.

For instance, a function call that generates the parent class:


```javascript
function f(phrase) {
  return class {
    sayHi() { alert(phrase); }
  };
}

class User extends f("Hello") {}

new User().sayHi(); // Hello
```

Here `class User` inherits from the result of `f("Hello")`.

That may be useful for advanced programming patterns when we use functions to generate classes depending on many conditions and can inherit from them.
````

----

## 📌 Inheritance of Static Properties in JavaScript

- **Static methods/properties** are defined on the **constructor (class itself)**, not on its `prototype`.
    
    ```js
    class Parent {
      static role = "Base";
      static sayHi() {
        return "Hello from Parent";
      }
    }
    ```
    
- **Child classes inherit static members** via the prototype chain of constructors:
    
    ```js
    class Child extends Parent {}
    
    console.log(Child.role);        // "Base"
    console.log(Child.sayHi());     // "Hello from Parent"
    ```
    
- Under the hood:
    
    - `Child.__proto__ === Parent`
        
    - `Child.prototype.__proto__ === Parent.prototype`
        
    - So `Child` can access static members through its `__proto__`.
        
- **Overriding**:
    
    ```js
    class Child extends Parent {
      static role = "Child";
    }
    
    console.log(Child.role);  // "Child"
    console.log(Parent.role); // "Base"
    ```
    

---

### 💪 Extending build-in classes

You can extend built-ins like `Array`, `Error`, `Map`, etc.

#### Example: Extending `Array`

```js
class MyArray extends Array {
  first() {
    return this[0];
  }
}

const arr = new MyArray(1, 2, 3);
console.log(arr.first());  // 1
console.log(arr instanceof Array);    // true
console.log(arr instanceof MyArray);  // true
```

- `MyArray` inherits everything from `Array`.
    
- Prototype chain:  
    `arr → MyArray.prototype → Array.prototype → Object.prototype`
    

#### 🚨 Very Interesting Thing: Methods Like `map`, `filter`, etc.

When you extend `Array`, built-in methods (`map`, `filter`, `slice`, etc.) **return new objects of the inherited type**, not plain `Array`.

```js
const doubled = arr.map(x => x * 2);

console.log(doubled instanceof MyArray); // ✅ true
console.log(doubled.first());            // works!
```

This is because many built-in classes in JS use an internal mechanism called **`Symbol.species`** to decide the constructor of derived objects.

- By default, `MyArray[Symbol.species]` is `MyArray`.
    
- You can override it if you want `map`, `filter`, etc. to return plain `Array` instead:
    

```js
class MyArray extends Array {
  static get [Symbol.species]() {
    return Array; // now derived methods return Array, not MyArray
  }
}

const arr = new MyArray(1, 2, 3);
const mapped = arr.map(x => x * 2);

console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array);   // true
```

#### Example: Extending `Error`

```js
class MyError extends Error {
  constructor(message) {
    super(message);
    this.name = "MyError";
  }
}

try {
  throw new MyError("Something went wrong!");
} catch (e) {
  console.log(e.name);    // "MyError"
  console.log(e.message); // "Something went wrong!"
  console.log(e.stack);   // stack trace
}
```

#### ⚠️ Gotchas

1. `Array` and some other built-ins are **exotic objects** with special behavior (like length auto-tracking).
    
2. Always call `super(...)` in your constructor if you override it.
    
3. Private fields (`#field`) still work, but subclasses can’t directly access a parent’s private field.
    
4. Built-in methods on subclasses often respect the **`Symbol.species`** property.
    

---

### ✅ Tips & Best Practices

- Use static methods for **utility functions** related to the class, not individual instances.
    
- Be cautious when **overriding static properties**: it shadows the parent’s value, doesn’t replace it globally.
    
- Remember: instances (`new Child()`) **cannot access static members** directly.
    