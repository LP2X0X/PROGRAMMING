---
tags:
  - html
  - term
  - fundamental
---

In HTML, an **attribute** provides **additional information** about an element. It always appears **in the opening tag**, and it’s written as a key-value pair.
![[Pasted image 20250804202246.png|500|center]]

---

### **✅ Basic Structure**

```html
<tagname attributeName="value">Content</tagname>
```

---

### **✅ Examples**

```html
<a href="https://example.com">Visit site</a>  <!-- href is the attribute -->
<img src="image.jpg" alt="An image">         <!-- src and alt are attributes -->
<input type="text" placeholder="Enter name"> <!-- type and placeholder -->
<button disabled>Click me</button>           <!-- disabled is a boolean attribute -->
```

---

### **✅ Attribute Types**

|**Type**|**Example**|**Notes**|
|---|---|---|
|**Standard**|href="...", src="...", class="..."|Most attributes are standard|
|**Boolean**|disabled, checked, readonly|Value is optional (e.g. <input disabled>)|
|**Global**|id, class, style, title|Can be used on **any** HTML element|
|**Event**|onclick, onmouseover, onchange|Used to bind JavaScript events|

---

### **🧠 Example with Multiple Attributes**

```html
<input type="email" id="user-email" placeholder="Enter your email" required>
```

This input has:

- type="email" → the input expects an email
    
- id="user-email" → used for JavaScript/CSS
    
- placeholder="..." → shows hint text
    
- required → makes it a mandatory field
    
