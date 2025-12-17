---
tags:
  - js
  - term
  - fundamental
---

**Event delegation** in JavaScript is a technique where you add a single event listener to a **parent element**, and use it to handle events triggered by its **child elements**. It relies on **[[Event Bubbling|event bubbling]]**, where events bubble up from the target element to the ancestors.

---

## **🔹 Why Use Event Delegation?**

✅ Better performance (fewer event listeners)
✅ Dynamically handle new child elements
✅ Clean, scalable code

---

## **🔹 Example: Without Delegation**

```js
document.getElementById("item1").addEventListener("click", () => {
  console.log("Item 1 clicked");
});
document.getElementById("item2").addEventListener("click", () => {
  console.log("Item 2 clicked");
});
// Repeated code for each item...
```

---

## **🔹 Example: With Event Delegation**

```html
<ul id="menu">
  <li>Home</li>
  <li>About</li>
  <li>Contact</li>
</ul>
```

```js
document.getElementById("menu").addEventListener("click", function (event) {
  if (event.target.tagName === "LI") {
    console.log("Clicked:", event.target.textContent);
  }
});
```

> 🔧 event.target is the actual element clicked.

> 🧠 You can use event.target.matches('selector') for more complex checks.

---

## **🔹 Dynamic Elements**

Event delegation works perfectly for elements added later:

```js
const menu = document.getElementById("menu");
const newItem = document.createElement("li");
newItem.textContent = "Blog";
menu.appendChild(newItem); // No new event listener needed
```

---

## **🔹 Example with event.target.closest()**

```js
menu.addEventListener("click", function (event) {
  const item = event.target.closest("li");
  if (item) {
    console.log("Clicked:", item.textContent);
  }
});
```

This is safer if there are nested elements inside \<li>.

---

## **🔸 Summary**

| **Concept**         | **Description**                           |
| ------------------- | ----------------------------------------- |
| event.target        | Actual clicked element                    |
| event.currentTarget | Element the event listener is attached to |
| matches(selector)   | Checks if element matches CSS selector    |
| closest(selector)   | Finds closest ancestor matching selector  |