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