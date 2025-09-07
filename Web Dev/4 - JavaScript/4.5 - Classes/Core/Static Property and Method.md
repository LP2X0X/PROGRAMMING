---
tags: 
 - js
 - static
---


- A **`static` method or property** belongs to the **class itself**, **not** to the instances created from that class.
    
- You call it on the class, not on `new` objects.
    

---

### 1. Static methods

```js
class Animal {
  static planet = "Earth";

  static compare(a, b) {
    return a.speed - b.speed;
  }
  
  // Same as this
  //Animal.compare = function () {
  // ...
  //}
}

let rabbit = { speed: 10 };
let turtle = { speed: 5 };

console.log(Animal.compare(rabbit, turtle)); // 5
console.log(Animal.planet); // "Earth"
```

- `compare` is called like `Animal.compare()`, not `rabbit.compare()`.
    
- `planet` belongs to the class, not the instance.
    

---

### 2. Static properties

```js
class Counter {
  static count = 0;

  constructor() {
    Counter.count++;
  }
}

let c1 = new Counter();
let c2 = new Counter();

console.log(Counter.count); // 2
```

- Each new object increments the **class-wide counter**.
    
- Instances don’t have `count` as their own property.
    

---

### 3. Inheritance with `static`

Static members are inherited by child classes:

```js
class Animal {
  static planet = "Earth";

  constructor(name, speed) {
    this.speed = speed;
    this.name = name;
  }

  run(speed = 0) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }

  static compare(animalA, animalB) {
    return animalA.speed - animalB.speed;
  }

}

// Inherit from Animal
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbits = [
  new Rabbit("White Rabbit", 10),
  new Rabbit("Black Rabbit", 5)
];

rabbits.sort(Rabbit.compare);

rabbits[0].run(); // Black Rabbit runs with speed 5.

alert(Rabbit.planet); // Earth
```

![[Pasted image 20250904171859.png|center|400]]

```ad-note
Detail explanation can be read [[extends Keyword#📌 Inheritance of Static Properties in JavaScript|here]].
```

---

### 4. Key difference from instance members

```js
class Animal {
  static greet() {
    return "Hello from class!";
  }
  sayHi() {
    return "Hi from instance!";
  }
}

console.log(Animal.greet());   // works
let dog = new Animal();
console.log(dog.sayHi());      // works
console.log(dog.greet());      // ❌ Error
```

---

### 5. No static inheritance in built-ins

- Built-in objects have their own static methods, for instance `Object.keys`, `Array.isArray` etc.

- As we already know, native classes extend each other. For instance, `Array` extends `Object`.

- Normally, when one class extends another, both static and non-static methods are inherited.

- But built-in classes are an exception. They don’t inherit statics from each other.

</br>

- For example, both `Array` and `Date` inherit from `Object`, so their instances have methods from `Object.prototype`. But `Array.[[Prototype]]` does not reference `Object`, so there’s no, for instance, `Array.keys()` (or `Date.keys()`) static method.
![[Pasted image 20250907184715.png|center|500]]


- As you can see, there’s no link between `Date` and `Object`. They are independent, only `Date.prototype` inherits from `Object.prototype`.

- That’s an important difference of inheritance between built-in objects compared to what we get with `extends`.

---

✅ **In short**:

- `static` makes methods/properties live on the **class**, not its instances.
    
- Good for utilities, counters, registries, or anything “class-wide.”
    
