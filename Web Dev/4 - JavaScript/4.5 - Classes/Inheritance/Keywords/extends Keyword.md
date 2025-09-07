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

### ✅ Tips & Best Practices

- Use static methods for **utility functions** related to the class, not individual instances.
    
- Be cautious when **overriding static properties**: it shadows the parent’s value, doesn’t replace it globally.
    
- Remember: instances (`new Child()`) **cannot access static members** directly.
    