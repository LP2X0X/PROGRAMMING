---
tags: js, distinguish, fundamental
---

## **📄 1. Definitions**

- 🔤 innerText: Returns the visible text of a node, considering CSS styles (e.g., display: none, visibility: hidden).
- 🧾 textContent: Returns all the text inside an element, regardless of CSS styles (including hidden text).

---

## **🔍 2. Key Differences**

|**Feature**|**innerText**|**textContent**|
|---|---|---|
|Includes hidden elements?|❌ No|✅ Yes|
|Reads rendered text?|✅ Yes|❌ No (reads raw DOM text)|
|Performance|⚠️ Slower (requires layout info)|✅ Faster (no layout required)|
|Includes  content?|❌ No|✅ Yes (if present as text)|
|Formatting (line breaks)|✅ Preserves visual line breaks|❌ Doesn’t preserve them|
|Reflow triggered?|✅ Yes|❌ No|

---

## **🧪 3. Example**

HTML:

```html
<div id="demo">
  Hello <span style="display: none">Hidden</span> World
</div>
```

JavaScript:

```js
const el = document.getElementById("demo");

console.log(el.innerText);   // "Hello World"
console.log(el.textContent); // "Hello Hidden World"
```

---

## **🧠 4. When to Use Which?**

|**Goal**|**Use**|
|---|---|
|Get only visible text|innerText|
|Get all text (including hidden content)|textContent|
|Better performance / raw content parsing|textContent|
|Text seen by users (e.g., screen readers)|innerText (sometimes)|

---

## **✅ Summary**

- Use textContent if you want the actual contents of the element, fast and raw.
- Use innerText if you care about what’s visible on screen, like for screen scraping or UI testing.
