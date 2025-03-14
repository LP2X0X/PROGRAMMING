- **Template literals** are literals delimited with backtick (`` ` ``) characters, allowing for [multi-line strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#multi-line_strings), [string interpolation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#string_interpolation) with embedded expressions, and special constructs called [tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates).

---

**📌 Basic Syntax**

```
const name = "Alice";
const message = `Hello, ${name}!`;
console.log(message); // Output: Hello, Alice!
```

  

---

**📚 Features of Template Literals**

---

**1. String Interpolation**

  

You can embed expressions using **${}** inside the template.

```
const a = 5;
const b = 10;
console.log(`Sum: ${a + b}`); // Output: Sum: 15
```

  

---

**2. Multi-line Strings**

  

No need to concatenate multiple lines with \n.

```
const paragraph = `
This is a multi-line
string using template literals.
`;
console.log(paragraph);
```

  

---

**3. Expression Evaluation**

  

You can embed any JavaScript expression inside **${}**.

```
const isActive = true;
console.log(`Status: ${isActive ? "Active" : "Inactive"}`);
```

  

---

**4. Function Calls**

  

You can call functions directly inside template literals.

```
const getName = () => "Bob";
console.log(`Hello, ${getName()}!`); // Output: Hello, Bob!
```

  

---

**5. Tagged Templates**

  

Use a **tag function** to process the template literal.

```
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => `${acc}<b>${values[i - 1] || ""}</b>${str}`);
}

const user = "Charlie";
console.log(highlight`Welcome, ${user}!`);
// Output: Welcome, <b>Charlie</b>!
```

  

---

**🧪 When to Use Template Literals**

• **Interpolating variables** and **expressions**.

• **Multi-line** strings without escape sequences.

• Creating **dynamic content** in applications.

• Implementing **custom string processing** with **tagged templates**.
