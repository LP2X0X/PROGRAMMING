---
tags: 
 - js
 - term
 - advance
---

## 🔹 The need for HomeObject

- First to say, from all that we’ve learned till now, it’s impossible for `super` to work at all!

- Yeah, indeed, let’s ask ourselves, how it should technically work? When an object method runs, it gets the current object as `this`. If we call `super.method()` then, the engine needs to get the `method` from the prototype of the current object. But how?

- The task may seem simple, but it isn’t. The engine knows the current object `this`, so it could get the parent `method` as `this.__proto__.method`. Unfortunately, such a “naive” solution won’t work.

- Let’s demonstrate the problem. Without classes, using plain objects for the sake of simplicity.

- In the example below, `rabbit.__proto__ = animal`. Now let’s try: in `rabbit.eat()` we’ll call `animal.eat()`, using `this.__proto__`:
	```js
	let animal = {
	  name: "Animal",
	  eat() {
	    alert(`${this.name} eats.`);
	  }
	};
	
	let rabbit = {
	  __proto__: animal,
	  name: "Rabbit",
	  eat() {
	    // that's how super.eat() could presumably work
	    this.__proto__.eat.call(this); // (*)
	  }
	};
	
	rabbit.eat(); // Rabbit eats.
	```

_ At the line `(*)` we take `eat` from the prototype (`animal`) and call it in the context of the current object. Please note that `.call(this)` is important here, because a simple `this.__proto__.eat()` would execute parent `eat` in the context of the prototype, not the current object.

- And in the code above it actually works as intended: we have the correct `alert`.

- Now let’s add one more object to the chain. We’ll see how things break:
	```js
	let animal = {
	  name: "Animal",
	  eat() {
	    alert(`${this.name} eats.`);
	  }
	};
	
	let rabbit = {
	  __proto__: animal,
	  eat() {
	    // ...bounce around rabbit-style and call parent (animal) method
	    this.__proto__.eat.call(this); // (*)
	  }
	};
	
	let longEar = {
	  __proto__: rabbit,
	  eat() {
	    // ...do something with long ears and call parent (rabbit) method
	    this.__proto__.eat.call(this); // (**)
	  }
	};
	
	longEar.eat(); // Error: Maximum call stack size exceeded
	```

- The code doesn’t work anymore! We can see the error trying to call `longEar.eat()`.

- It may be not that obvious, but if we trace `longEar.eat()` call, then we can see why. In both lines `(*)` and `(**)` the value of `this` is the current object (`longEar`). That’s essential: all object methods get the current object as `this`, not a prototype or something.

- So, in both lines `(*)` and `(**)` the value of `this.__proto__` is exactly the same: `rabbit`. They both call `rabbit.eat` without going up the chain in the endless loop.

- Here’s the picture of what happens:
![[Pasted image 20250904095446.png|center|400]]

1. Inside `longEar.eat()`, the line `(**)` calls `rabbit.eat` providing it with `this=longEar`.
	```js
    // inside longEar.eat() we have this = longEar
    this.__proto__.eat.call(this) // (**)
    // becomes
    longEar.__proto__.eat.call(this)
    // that is
    rabbit.eat.call(this);
    ```

</br>

2. Then in the line `(*)` of `rabbit.eat`, we’d like to pass the call even higher in the chain, but `this=longEar`, so `this.__proto__.eat` is again `rabbit.eat`!
	```js
	// inside rabbit.eat() we also have this = longEar
	this.__proto__.eat.call(this) // (*)
	// becomes
	longEar.__proto__.eat.call(this)
	// or (again)
	rabbit.eat.call(this);
	```

</br>

3. …So `rabbit.eat` calls itself in the endless loop, because it can’t ascend any further.

> The problem can’t be solved by using `this` alone.

---

## 🔹 What is the HomeObject?

- The **HomeObject** is an **internal reference** that is created for methods defined in object literals or class syntax.
    
- It stores **the object where the method was originally defined**.
    
- This is what allows the `super` keyword to work, since `super` needs to know "where to start looking in the prototype chain."
    

---

## 🔹 Example with a class

```js
class Parent {
  greet() {
    console.log("Hello from Parent");
  }
}

class Child extends Parent {
  greet() {
    super.greet(); // uses HomeObject to find Parent
    console.log("Hello from Child");
  }
}

const c = new Child();
c.greet();
// Hello from Parent
// Hello from Child
```

Here:

- The method `greet` inside `Child` has a `[[HomeObject]]` pointing to `Child.prototype`.
    
- When we write `super.greet()`, JS looks at the `HomeObject` (Child.prototype), then walks up the prototype chain to find `Parent.prototype.greet`.
    

---

## 🔹 Example with object literals

```js
const parent = {
  greet() {
    console.log("Hello from parent");
  }
};

const child = {
  greet() {
    super.greet();
    console.log("Hello from child");
  }
};

Object.setPrototypeOf(child, parent);

child.greet();
// Hello from parent
// Hello from child
```

Here:

- `child.greet` has a `[[HomeObject]]` pointing to `child`.
    
- That’s why `super.greet()` works — it knows to look at `child`’s prototype (`parent`) for `greet`.
    

---

## 🔹 Important notes

1. **Only methods defined using shorthand syntax or in classes get a HomeObject.**
    
    ```js
    const obj = {
      // ✅ method shorthand → has HomeObject
      method() {},
    
      // ❌ arrow function or function property → no HomeObject
      func: function () {},
      arrow: () => {}
    };
    ```
    
    → This is why `super` only works inside real methods, not arrow functions or function expressions.
    
2. **HomeObject is hidden.**
    
    - You cannot access it in JavaScript code.
        
    - It exists only in the spec to make `super` possible.
        

---

✅ **In short:**  
The **HomeObject** is an internal hidden property that “anchors” a method to the object or class where it was created, so that `super` knows which prototype to use.

---

Would you like me to also diagram how **HomeObject interacts with `super` step by step in a prototype chain lookup**?