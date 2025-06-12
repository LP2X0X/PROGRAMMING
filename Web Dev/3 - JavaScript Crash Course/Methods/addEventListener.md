---
tags: js, method, fundamental
---

The addEventListener() method in JavaScript is used to attach event handlers to DOM elements — meaning you can make elements respond to user actions like clicks, typing, hovering, and more.

---

## **✅ Basic Syntax**

```js
element.addEventListener(eventType, handlerFunction, useCapture);
```

- eventType → a string like “click”, “keydown”, “mouseover”, etc.
- handlerFunction → the function to run when the event happens
- useCapture → optional (true/false); default is false (bubble phase)

---

## **🔹 Example**

```html
<button id="myBtn">Click me</button>
```

```js
const button = document.getElementById("myBtn");

button.addEventListener("click", function () {
  alert("Button clicked!");
});
```

---

## **🔄 Reusable Function Example**

```js
function greet() {
  console.log("Hello!");
}

button.addEventListener("click", greet);
```

---

## **🧹 Removing an Event Listener**

You can remove an event listener using removeEventListener — but only if the handler function is named (not anonymous):

```js
function greet() {
  console.log("Hello!");
}

button.addEventListener("click", greet);
button.removeEventListener("click", greet); // must match exactly
```

---

## **📍 Multiple Listeners Allowed**

You can add multiple event listeners for the same event type on the same element:

```js
button.addEventListener("click", () => console.log("First!"));
button.addEventListener("click", () => console.log("Second!"));
```

---

## **🧠 Event Object**

Your handler can accept an event parameter:

```js
button.addEventListener("click", function (event) {
  console.log("Clicked at:", event.clientX, event.clientY);
});
```

- This object contains info like:
	- event.target → the clicked element
	- event.type → the type of event
	- event.preventDefault() → prevents default action (e.g., form submit)

---

## **✅ Common Event Types**

|**Event**|**Triggered When…**|
|---|---|
|click|Element is clicked|
|mouseover|Cursor enters element|
|keydown|Key is pressed down|
|submit|Form is submitted|
|input|Input field value changes|
|load|Page or image has loaded|
