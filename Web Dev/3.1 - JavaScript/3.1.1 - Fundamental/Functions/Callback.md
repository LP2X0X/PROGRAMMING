---
tags: js, term, fundamental
---

### **🔁 Callback in JavaScript**

A **callback** is a **function passed as an argument** to another function, which is then **executed later**—either **after a task completes**, or **in response to an event**.

---

### **✅ Basic Example**

```js
function greet(name, callback) {
  console.log("Hi, " + name);
  callback();
}

function sayBye() {
  console.log("Bye!");
}

greet("Alice", sayBye); // Here we pass the call back function without the parentheses
```

➡️ Output:

```
Hi, Alice
Bye!
```

- sayBye is the **callback** function passed to greet.

---

### **🕐 Why Use Callbacks?**

- Handle **asynchronous operations** (e.g., loading files, making API calls).
- React to **events** like clicks, form submissions, etc.
- Customize behavior dynamically.

---

### **🧭 Asynchronous Callback Example**

```js
setTimeout(() => {
  console.log("This runs after 2 seconds");
}, 2000);
```

- setTimeout uses a **callback function** that runs after the delay.

---

### **📦 Custom Async Function with Callback**

```js
function fetchData(callback) {
  setTimeout(() => {
    const data = "Some data";
    callback(data);
  }, 1000);
}

fetchData(function(result) {
  console.log("Received:", result);
});
```

---

### **⚠️ Callback Hell (Nested Callbacks)**

```js
doSomething(function(result1) {
  doSomethingElse(result1, function(result2) {
    doAnotherThing(result2, function(result3) {
      // gets messy fast!
    });
  });
});
```

Use **Promises** or **async/await** to fix this.

---

### **✅ Summary**

|**Term**|**Meaning**|
|---|---|
|Callback|Function passed to another function|
|Sync/Async|Can be used in both types of operations|
|Key use|Delayed execution, event handling, async|