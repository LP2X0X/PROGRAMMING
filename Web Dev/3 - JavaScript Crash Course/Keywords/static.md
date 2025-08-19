---
tags:
  - js
  - keyword
  - fundamental
---

In JavaScript, the static keyword is used inside classes to define **static methods** or **static properties**. These belong to the **class itself**, **not to instances** of the class.

### **🔹 Syntax**

```js
class MyClass {
  static staticMethod() {
    console.log("This is a static method.");
  }

  static staticProperty = "I'm static!";
}
```

---

### **🔹 Usage**
#### **✅ Accessing a Static Method**

```js
MyClass.staticMethod(); // ✅ Works
const obj = new MyClass();
// obj.staticMethod(); // ❌ Error: not a function
```

#### **✅ Accessing a Static Property**

```js
console.log(MyClass.staticProperty); // ✅ Works
```

---

### **🔹 Why Use static ?**

1. **Utility functions** that don’t depend on instance state:

```js
class MathUtil {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtil.add(2, 3)); // 5
```

2. **Constants or configuration values**

```js
class Config {
  static VERSION = "1.0.0";
}

console.log(Config.VERSION); // "1.0.0"
```

---

### **🔸 Notes**

- static methods can’t access this in the way instance methods do.
- They can be called without creating an object.
- Static members **can be inherited** by child classes.
    

```js
class Parent {
  static greet() {
    return "Hello from Parent";
  }
}

class Child extends Parent {}

console.log(Child.greet()); // "Hello from Parent"
```