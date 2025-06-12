---
tags: js, term, fundamental
---

🧠 Event bubbling is a core concept in the JavaScript event model. It describes how events move (or “bubble”) through the DOM tree when a user interacts with a webpage — especially when clicking, typing, or interacting with elements.

---

## **🧱 What is Event Bubbling?**

When an event occurs on an element, it doesn’t stop there — it “bubbles up” from the target element through its ancestors, all the way up to the document.

For example:

```html
<div id="parent">
  <button id="child">Click Me</button>
</div>
```

If a click happens on the button:

1. The event triggers on the button (#child)
2. Then it bubbles up to the parent (#parent)
3. Then up to \<body>, \<html>, and document

---

## **📌 Example: Event Bubbling in Action**

```js
document.getElementById("child").addEventListener("click", () => {
  console.log("Child clicked");
});

document.getElementById("parent").addEventListener("click", () => {
  console.log("Parent clicked");
});
```

🖱️ When you click the button, you’ll see:

```
Child clicked
Parent clicked
```

The event starts at the child and bubbles to the parent.

---

## **✋ Preventing Bubbling**

You can stop an event from bubbling using:

```js
event.stopPropagation();
```

Example:

```js
document.getElementById("child").addEventListener("click", (e) => {
  console.log("Child only");
  e.stopPropagation(); // prevents parent handler from firing
});
```

---

## **🔁 Event Phases (Advanced)**

JavaScript events go through three phases:

1. **Capture Phase** (down from root to target)
2. **Target Phase** (event reaches the actual element)
3. **Bubble Phase** (up from target to root)

By default, addEventListener listens during bubbling. You can listen during the capture phase like this:

```js
element.addEventListener("click", handler, true); // true = capture phase
```

---

## **✅ Why Use Bubbling?**

- Simplifies code with event delegation (e.g., listen on a parent instead of each child)
- Improves performance by reducing the number of event listeners
