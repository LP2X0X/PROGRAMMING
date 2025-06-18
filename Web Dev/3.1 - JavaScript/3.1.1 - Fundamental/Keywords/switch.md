---
tags: js, keyword, fundamental
---

The switch statement in JavaScript is a **control flow structure** used to compare a value against multiple possible cases and execute corresponding code.

---

## **🧱 Syntax**

```js
switch (expression) {
  case value1:
    // code for value1
    break;
  case value2:
    // code for value2
    break;
  default:
    // code if no case matches
}
```

---

## **✅ Example**

```js
const day = 3;

switch (day) {
  case 1:
    console.log("Monday");
    break;
  case 2:
    console.log("Tuesday");
    break;
  case 3:
    console.log("Wednesday");
    break;
  default:
    console.log("Another day");
}
```

### **Output:**

```
Wednesday
```

---

## **⚠️ Don’t Forget break**

Without break, execution continues into the next case (“fall-through”):

```js
const color = "green";

switch (color) {
  case "red":
    console.log("Stop");
  case "green":
    console.log("Go");
  case "yellow":
    console.log("Caution");
}
```

### **Output:**

```
Go
Caution
```

To fix this, add break statements:

```
switch (color) {
  case "green":
    console.log("Go");
    break;
}
```

---

## **🧠 Use default for unmatched cases**

```js
switch (command) {
  case "start":
    runApp();
    break;
  case "stop":
    stopApp();
    break;
  default:
    console.log("Unknown command");
}
```

---

## **🔁 Grouping Cases**

```js
const value = 0;

switch (value) {
  case 0:
  case false:
  case null:
    console.log("Falsy value");
    break;
}
```

---

## **✅ When to Use switch**

- When you’re comparing **one variable** against **many fixed values**
- When **readability** is better than a long if-else if chain