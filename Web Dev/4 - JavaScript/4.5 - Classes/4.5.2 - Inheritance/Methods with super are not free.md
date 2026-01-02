---
tags: 
 - js
 - note
 - advance
---

- In JavaScript, **functions are typically “free”**—they aren’t permanently tied to any particular object. You can copy a plain function around and execute it with different `this` contexts.
    
- **But methods defined with `super` are not free.** When you define a method using syntax like `eat() { ... }`, the method gets a hidden internal binding called **`[[HomeObject]]`**, which permanently anchors it to the object where it was originally defined.
    
- This means methods that use `super` **remember their home object**, and cannot function correctly if transferred to another object—unlike free-standing functions.
    
- For example:
    
    ```js
    let rabbit = {
      __proto__: animal,
      sayHi() {
        super.sayHi();
      }
    };
    
    let tree = {
      __proto__: plant,
      sayHi: rabbit.sayHi  // copied method
    };
    
    tree.sayHi(); // Unexpectedly calls animal.sayHi(), not plant.sayHi
    ```
    
    The call still references the `rabbit` prototype chain because `[[HomeObject]]` remains bound to `rabbit`.
    
- **Important detail**: `[[HomeObject]]` is only set for methods defined using the method shorthand (`method() { ... }`), not for function-valued properties like `method: function() { ... }`. In the latter case, `super` will not work at all.
    

![[Pasted image 20250904155454.png|center|400]]

---

**In summary**: Unlike normal free functions, methods using `super` have their origin baked in. They're tied to the class or object where they're declared, and that binding **cannot be changed**.

> So it’s not safe to copy a method with `super` from one object to another.

