---
tags: 
  - js
  - class
  - term
  - fundamental
---

In JavaScript, a **class expression** is a way to define a class using an expression, rather than a declaration.

### Syntax

```js
// unnamed class expression
let MyClass = class {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hello, ${this.name}`;
  }
};

// named class expression
let YourClass = class NamedClass {
  constructor(age) {
    this.age = age;
  }
  getAge() {
    return this.age;
  }
};
```

### Key Points

1. **Anonymous vs. Named**
    
    - **Anonymous class expression**: No name after `class`.
        
    - **Named class expression**: Has a name, but the name is only visible inside the class body (useful for recursion/debugging).
        
    
    ```js
    let User = class UserHelper {
      constructor(id) {
        this.id = id;
      }
      getId() {
        // we can refer to UserHelper here, but not outside
        return this.id;
      }
    };
    
    console.log(typeof UserHelper); // ReferenceError
    ```
    
2. **First-class citizens**  
    Since classes are expressions, you can:
    
    - Assign them to variables.
        
    - Pass them as arguments to functions.
        
    - Return them from functions.
        
    
    ```js
    function createClass(greeting) {
      return class {
        sayHello(name) {
          return `${greeting}, ${name}`;
        }
      };
    }
    
    let Greeter = createClass("Hi");
    let g = new Greeter();
    console.log(g.sayHello("Alice")); // Hi, Alice
    ```
    
3. **Difference from class declaration**
    
    - **Class declaration** must have a name:
        
        ```js
        class Animal {}
        ```
        
    - **Class expression** can be anonymous:
        
        ```js
        let Animal = class {};
        ```
        

Would you like me to also break down the **practical cases where class expressions are better than class declarations** (like inside functions or dynamic class creation)?