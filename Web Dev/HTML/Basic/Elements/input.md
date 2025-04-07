The \<input> element in HTML is used to **create interactive form controls** that accept user input. It’s super versatile and used in forms for everything from text fields to checkboxes.

---

**📌 Basic Syntax:**

```html
<input type="text" name="username" placeholder="Enter your username">
```

  

---

**🔑 Common Attributes:**

|**Attribute**|**Description**|
|---|---|
|type|**(Required)** Defines the input type: text, email, password, checkbox, radio, submit, etc.|
|name|Identifies the input when submitting form data.|
|value|Sets a default value.|
|placeholder|A short hint displayed inside the field.|
|required|Makes the input mandatory.|
|disabled|Makes the input uneditable and gray.|
|readonly|Prevents editing but still sends the value on submit.|
|maxlength / minlength|Limits the number of characters.|
|min / max|For number/date types — sets a range.|
|step|Defines the interval for number inputs.|
|autocomplete|Controls whether the browser can auto-fill.|

  

---

**✅ Example Types:**

```html
<input type="text" placeholder="Text field"><br>
<input type="email" placeholder="Email"><br>
<input type="password" placeholder="Password"><br>
<input type="checkbox"> Remember me<br>
<input type="radio" name="gender" value="male"> Male<br>
<input type="radio" name="gender" value="female"> Female<br>
<input type="submit" value="Submit">
```

  

---

**💡 Tips:**

• Always use label elements for accessibility.

• Use name attributes when inputs are inside a /<form> to make sure data gets submitted.

• Combine with pattern attribute for custom validation (e.g., regex).

  

Want a breakdown of specific input types or how to style inputs with CSS?