---
tags: 
 - js
 - note
 - advance
---

## 🔹 Step 1: What happens with a **method**

```js
class Button {
  constructor(value) {
    this.value = value;
  }

  click() {          // method
    console.log(this.value);
  }
}

let b = new Button("hello");

// Pass method as callback
setTimeout(b.click, 1000);
```

👉 Here’s what happens:

1. `b.click` is **looked up** on `b`. It’s found on `Button.prototype`.
    
2. That function is **copied as a reference** into `setTimeout`.
    
    ```js
    let f = b.click;   // f = function click() { console.log(this.value) }
    ```
    
3. `setTimeout` later calls `f()` as a **plain function**.
    
    - No object before the dot.
        
    - So `this` falls back to the default (undefined in strict mode, or global in sloppy mode).
        
4. Result: `this.value` is lost.
    

---

## 🔹 Step 2: What happens with a **class field (arrow function)**

```js
class Button {
  constructor(value) {
    this.value = value;
  }

  click = () => {    // class field
    console.log(this.value);
  }
}

let b = new Button("hello");
setTimeout(b.click, 1000);
```

👉 Here’s the key difference:

- `click = () => {}` is **not on the prototype**. It’s created **inside the constructor**, as a **property of the instance**:
    
    ```js
    this.click = () => { console.log(this.value); };
    ```
    
- Arrow functions don’t have their own `this`.  
    Instead, they “close over” the `this` value of the surrounding scope (the constructor call).  
    At that moment, `this` = the new instance (`b`).
    
- So `this` inside the arrow function is permanently bound to the instance.
    

Even if you pass `b.click` around:

```js
let f = b.click;
f(); // still prints "hello"
```

…it still works, because the arrow function has already locked in the correct `this`.

---

## 🔹 Step 3: The Core Contrast

- **Method (prototype):**
    
    - Lives on `Button.prototype`.
        
    - Each call decides `this` based on _how_ it’s called.
        
    - Passing it around loses context unless you manually bind it.
        
- **Class field (arrow function):**
    
    - Lives directly on the instance.
        
    - Arrow function captures `this` from the constructor, so it’s forever tied to that instance.
        
    - Passing it around never loses context.
        

---

✅ **In short:**

- Methods are “free functions” sitting on the prototype, and `this` is determined at call time.
    
- Arrow function fields are created per instance, and `this` is locked to the instance at construction time.
    

---

```ad-note
**Here's my own explanation:**
Prototype methods are stored on the class’s prototype, so all instances "have" them. The trade-off is that when you pass a prototype method around, it doesn’t automatically know which instance it came from — the `this` context can be lost.

By contrast, methods defined as class fields are created directly on each instance. Since they capture the instance’s `this` at creation time, they always know which object they belong to and never lose their context.
```