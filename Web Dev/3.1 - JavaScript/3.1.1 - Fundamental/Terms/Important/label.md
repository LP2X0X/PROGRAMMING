---
tags:
  - js
  - term
  - fundamental
---

- In JavaScript, a **label** is a named identifier followed by a colon (:) that can be used with **break** or **continue** to control flow in **nested loops**.

---

## **🔖 Syntax**

```js
labelName: 
for (...) {
  // code
  break labelName;
}
```

---

## **✅ Example: Breaking out of nested loops**

```js
outerLoop: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    console.log(`i=${i}, j=${j}`);
    if (i === 1 && j === 1) {
      break outerLoop; // breaks the outer loop
    }
  }
}
```

### **Output:**

```
i=0, j=0
i=0, j=1
i=0, j=2
i=1, j=0
i=1, j=1  ← breaks both loops here
```

---

## **✅ Example: Continuing outer loop from inner loop**

```js
mainLoop: for (let x = 1; x <= 3; x++) {
  for (let y = 1; y <= 3; y++) {
    if (y === 2) continue mainLoop;
    console.log(`x=${x}, y=${y}`);
  }
}
```

### **Output:**

```
x=1, y=1
x=2, y=1
x=3, y=1
```

---

## **⚠️ When to use**


Use **labels**:

- When you have **deeply nested loops**
- And need to **break or continue** an **outer loop** from **inside an inner loop**


✅ **Avoid labels** if you can restructure your code more cleanly—labels are powerful but can hurt readability if overused.