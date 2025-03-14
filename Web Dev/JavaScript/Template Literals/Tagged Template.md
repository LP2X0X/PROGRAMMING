**📌 Tagged Templates in JavaScript**

  

A **tagged template** allows you to **customize** how template literals are processed by using a **function** before the template. This gives you control over the content inside the template literal.

---

**✅ Basic Syntax**

```
function tagFunction(strings, ...values) {
  console.log(strings); // Array of literal strings
  console.log(values);  // Array of evaluated expressions
}

const name = "Alice";
tagFunction`Hello, ${name}!`;
```

**Output:**

```
[ 'Hello, ', '!' ]
[ 'Alice' ]
```

  

---

**📚 How Tagged Templates Work**

  

When you use a tag function:

1. **strings** – Array of static string **literals**.

2. **values** – Array of **interpolated** expressions.

---

**🎯 Example 1: Custom Formatter**

```
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => `${result}${str}<b>${values[i] || ''}</b>`, '');
}

const user = "Bob";
console.log(highlight`Hello, ${user}!`);
```

**Output:**

```
Hello, <b>Bob</b>!
```

  

---

**🎯 Example 2: Escaping HTML (Prevent XSS)**

```
function escapeHTML(strings, ...values) {
  const escape = (str) =>
    str.replace(/</g, "&lt;").replace(/>/g, "&gt;");

  return strings.reduce((result, str, i) =>
    result + str + (values[i] ? escape(values[i]) : ""), "");
}

const userInput = "<script>alert('XSS');</script>";
console.log(escapeHTML`User input: ${userInput}`);
```

**Output:**

```
User input: &lt;script&gt;alert('XSS');&lt;/script&gt;
```

  

---

**🎯 Example 3: Localization (i18n)**

```
function i18n(strings, ...values) {
  const translations = {
    "Hello, ": "Hola, ",
    "!": "!"
  };

  return strings.reduce((result, str, i) =>
    result + (translations[str] || str) + (values[i] || ""), "");
}

const name = "Carlos";
console.log(i18n`Hello, ${name}!`);
```

**Output:**

```
Hola, Carlos!
```

  

---

**🔍 When to Use Tagged Templates**

• **Custom formatting** (e.g., bold/highlight).

• **Sanitizing** user input (e.g., prevent XSS).

• **Localization (i18n)** for multi-language support.

• **Logging** with custom output.
