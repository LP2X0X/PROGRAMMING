---
tags: 
 - js
 - object
 - function
 - property
---

## **Function as a Property of an Object in JavaScript**

In JavaScript, **functions can be properties of objects**. When a function is a property of an object, it’s called a **method**.

### **1. Defining a Function Property (Method)**

You can define a function inside an object in two main ways:

**a) Traditional function syntax**

```javascript
const person = {
    name: "Alice",
    greet: function() {
        console.log("Hello, " + this.name);
    }
};

person.greet(); // Output: Hello, Alice
```

**b) ES6 shorthand method syntax**

```javascript
const person = {
    name: "Bob",
    greet() {
        console.log("Hi, " + this.name);
    }
};

person.greet(); // Output: Hi, Bob
```

---

### **2. Accessing a Function Property**

- Use **dot notation**: `object.method()`
    
- Use **bracket notation**: `object["method"]()`
    

```javascript
person["greet"](); // Output: Hi, Bob
```

---

### **3. `this` Keyword Inside Method**

- `this` refers to the **object that owns the method**.
    

```javascript
const car = {
    brand: "Toyota",
    getBrand() {
        console.log(this.brand);
    }
};

car.getBrand(); // Output: Toyota
```

---

### **4. Function Properties Can Be Assigned Dynamically**

You can add a function to an object even after it is created:

```javascript
const dog = { name: "Max" };

dog.bark = function() {
    console.log(this.name + " says Woof!");
};

dog.bark(); // Output: Max says Woof!
```

---

### **5. Arrow Functions as Object Methods**

- Arrow functions **do not have their own `this`**. `this` refers to the **enclosing lexical scope**, not the object.
    

```javascript
const cat = {
    name: "Whiskers",
    meow: () => {
        console.log(this.name);
    }
};

cat.meow(); // Output: undefined
```

- **Tip:** Use traditional function syntax if you need `this` to refer to the object.
    

---

### **6. Example Combining Multiple Methods**

```javascript
const calculator = {
    a: 5,
    b: 3,
    add() {
        return this.a + this.b;
    },
    multiply() {
        return this.a * this.b;
    }
};

console.log(calculator.add());      // 8
console.log(calculator.multiply()); // 15
```

---

✅ **Key Points to Remember:**

1. Functions can be properties of objects → called **methods**.
    
2. Access methods using dot or bracket notation.
    
3. `this` inside a method refers to the **object itself**.
    
4. Arrow functions **should not be used** as object methods if you need `this`.
    
5. Methods can be added dynamically to objects.
    