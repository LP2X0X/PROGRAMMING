---
tags: 
 - js
 - class
 - inheritance
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