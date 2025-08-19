---
tags:
  - js
  - loop
  - fundamental
---

A **for** loop in JavaScript is a control structure used to repeat a block of code a certain number of times. It’s commonly used when you know how many times you want to loop.

---

## **🔁 Basic Syntax**

```js
for (initialization; condition; update) {
  // Code to run each loop
}
```

- 🟢 initialization – runs once at the beginning
- 🔄 condition – checked before each loop; if false, the loop ends
- ⏫ update – runs after each iteration

---

## **🧪 Example: Count from 1 to 5**

```js
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
// Output: 1 2 3 4 5
```

---

## **📦 Example: Loop through an array**

```js
const fruits = ["apple", "banana", "cherry"];

for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}
// Output: apple banana cherry
```

---

## **💡 Other Loop Types for Comparison**

|**Type**|**When to Use**|
|---|---|
|for loop|Fixed number of iterations|
|for…of|Loop over values of iterable (e.g. arrays)|
|for…in|Loop over keys/indexes (objects or arrays)|
|while loop|Repeat while a condition is true (unspecified #)|
|do…while|Like while, but runs at least once|

---

## **🔄 Example with continue and break**

```js
for (let i = 0; i < 10; i++) {
  if (i === 3) continue; // skip this iteration
  if (i === 7) break;    // exit the loop
  console.log(i);
}
// Output: 0 1 2 4 5 6
```



