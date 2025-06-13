---
tags: js, term, fundamental
---

- A template literal **is a special kind of string** that can **evaluate any expressions** embedded within it.
- This gives you the flexibility to dynamically populate a string with the values of variables, the results of calculations, or other code, instead of having to type out every character of the string exactly or combine several variables into a string with the + operator.
  
---

### How to create
- Template literals are enclosed in backticks (\`) instead of quotation marks.
- You incorporate code using *placeholder syntax*, which look like this: ${ }.

```js
let name = "Nick";
`Hello ${name}!`;
-> `Hello Nick!`
`There are ${60 * 60 * 24} seconds in a day`;
-> `There are 86400 seconds in a day`
```

---

## 📚 Features of Template Literals

---

### **1. String Interpolation**

You can embed expressions using **${}** inside the template.

```js
const a = 5;
const b = 10;
console.log(`Sum: ${a + b}`); // Output: Sum: 15
```

  

---

### **2. Multi-line Strings**

No need to concatenate multiple lines with \n.

```js
const paragraph = `
This is a multi-line
string using template literals.
`;
console.log(paragraph);
```

---

### **3. Expression Evaluation**

You can embed any JavaScript expression inside **${}**.

```js
const isActive = true;
console.log(`Status: ${isActive ? "Active" : "Inactive"}`);
```

---

### **4. Function Calls**

You can call functions directly inside template literals.

```js
const getName = () => "Bob";
console.log(`Hello, ${getName()}!`); // Output: Hello, Bob!
```

---

### **5. Tagged Templates**
  
Use a **tag function** to process the template literal.

```js
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => `${acc}<b>${values[i - 1] || ""}</b>${str}`);
}

const user = "Charlie";
console.log(highlight`Welcome, ${user}!`);
// Output: Welcome, <b>Charlie</b>!
```

---

### **🧪 When to Use Template Literals**
- **Interpolating variables** and **expressions**.
- **Multi-line** strings without escape sequences.
- Creating **dynamic content** in applications.
- Implementing **custom string processing** with **tagged templates**.
