---
tags: 
 - js
 - keyword
 - fundamental
---

When you write something like:

```js
const person = new Person("Alice");
```

JavaScript does **four things** behind the scenes:

1. **Create a new empty object**
    
    ```js
    {}
    ```
    
    A brand-new object is created in memory.
    
2. **Set the prototype**
    
    - The new object’s internal `[[Prototype]]` is linked to `Person.prototype`.
        
    
    ```js
    Object.setPrototypeOf(obj, Person.prototype);
    ```
    
3. **Bind `this`**
    
    - Inside the constructor, `this` refers to the **newly created object**.
        
    - The constructor function runs with that binding.
        
    
    ```js
    function Person(name) {
      this.name = name; // 'this' = the new object
    }
    ```
    
4. **Return the object**
    
    - If the constructor doesn’t explicitly return something, the new object is returned automatically.
        
    - If it does return an object, that returned object replaces the new one.
        

---

### Example:

```js
function Person(name) {
  this.name = name;
  this.sayHi = function () {
    console.log("Hi, I’m " + this.name);
  };
}

const user = new Person("Alice");
user.sayHi(); // "Hi, I’m Alice"
```

What happened internally:

1. `{}` created
    
2. Linked to `Person.prototype`
    
3. `this` → new object
    
4. `name` and `sayHi` added
    
5. Object returned
    

---

### 💡 Without `new`

If you forget to use `new`:

```js
const user = Person("Alice");
```

- `this` will refer to the **global object** (`window` in browsers, `global` in Node).
    
- No object is returned (unless you manually return one).
    
- It can cause bugs or unexpected behavior.
    

---

### 🧬 With Classes

Under the hood, `class` syntax is just syntactic sugar around this same concept:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

const user = new Person("Alice");
```

→ Works the same as the constructor function version.

---

### 🧠 Summary

|Step|Description|
|---|---|
|1|Creates a new empty object|
|2|Links it to the constructor’s prototype|
|3|Calls the constructor with `this` bound to the new object|
|4|Returns the new object (unless another object is explicitly returned)|