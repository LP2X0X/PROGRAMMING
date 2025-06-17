---
tags: html, element, fundamental
---

In HTML and JavaScript, a label is used to provide a user-friendly description for form controls (like inputs, checkboxes, radios, etc.). It improves usability and accessibility, especially for screen readers.

---

## **✅ Basic Syntax**

```html
<label for="email">Email Address</label>
<input type="email" id="email" />
```

- The for attribute in label must match the id of the input element.
- Clicking the label will focus the associated input field — very useful for UX.

---

## **📦 Implicit Label (Nested)**

Alternatively, you can nest the input inside the label:

```html
<label>
  Email Address
  <input type="email" />
</label>
```

This links the label and input without needing the for/id attributes.

---

## **🧠 JavaScript Notes**

- You can select a label like any other element:

```js
const label = document.querySelector("label[for='email']");
```

- You can change the text using:

```js
label.textContent = "Updated Email:";
```

---

## **🎯 Why Use ?**

- ✅ Increases accessibility for screen readers
- ✅ Makes forms easier to use (click label = focus input)
- ✅ Helps with form styling and UX
