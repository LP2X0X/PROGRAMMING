---
tags: 
 - js
 - class
 - inheritance
 - override
---

## 🔹 What is method overriding?

When a **child class** defines a method with the **same name** as a method in the **parent class**, the child’s version takes precedence for instances of the child.

---

## 🔹 Example

```js
class Animal {
  speak() {
    console.log("Generic animal sound");
  }
}

class Dog extends Animal {
  speak() {
    console.log("Woof!");
  }
}

const a = new Animal();
const d = new Dog();

a.speak(); // Generic animal sound
d.speak(); // Woof!  <-- overrides Animal's speak
```

---

## 🔹 Key points

1. **Overriding happens at the prototype chain level**
    
    - `Dog.prototype.speak` replaces `Animal.prototype.speak` for dog instances.
        
    - But `Animal.prototype.speak` still exists; dog instances just never reach it, since the method name already matches at `Dog.prototype`.
        

---

## 🔹 Prototype chain visualization

```js
d (instance)
   ⬆ __proto__
Dog.prototype { speak: "Woof!" }  ← overrides
   ⬆ __proto__
Animal.prototype { speak: "Generic animal sound" }
   ⬆ __proto__
Object.prototype
```

---

## 🔹 Important

- Only **methods with the same name** are overridden.
    
- Other parent methods are still available.
    
    ```js
    class Cat extends Animal {
      meow() { console.log("Meow"); }
    }
    
    const c = new Cat();
    c.meow();   // Meow
    c.speak();  // Generic animal sound (not overridden here)
    ```
    

---

👉 So, overriding in JS just means **redefining a method in the child’s prototype with the same name**.