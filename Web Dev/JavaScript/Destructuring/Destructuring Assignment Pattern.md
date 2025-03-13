### Assignment Pattern
- In assignment patterns, the pattern does not start with a keyword. Each destructured property is assigned to a target of assignment — which may either be declared beforehand with `var` or `let`, or is a property of another object — in general, anything that can appear on the left-hand side of an assignment expression.

```js
const numbers = [];
const obj = { a: 1, b: 2 };
({ a: numbers[0], b: numbers[1] } = obj);
// The properties `a` and `b` are assigned to properties of `numbers`
```

---

## Side note
The **parentheses** around this assignment in JavaScript are required due to how the JavaScript parser interprets code. Let’s break down why.

**📌 Why Are Parentheses Needed?**

When you use **object destructuring assignment** outside of a declaration (like const, let, or var), the parser may confuse it with a **block statement**.

Without parentheses:

```
{ a: numbers[0], b: numbers[1] } = obj; // ❌ SyntaxError
```

✅ **With parentheses:**

```
({ a: numbers[0], b: numbers[1] } = obj); // ✅ Correct syntax
```

**📖 Explanation:**

• **Without parentheses**: JavaScript interprets the {} as the start of a **block scope** (like in an if or for statement), not as an object.

• **With parentheses**: It tells the parser to treat the statement as an **expression** (not a block), allowing **destructuring assignment**.


**📊 When Do You Need Parentheses?**

1. **When destructuring without a const, let, or var:**

```
let obj = { a: 1, b: 2 };
let numbers = [];

({ a: numbers[0], b: numbers[1] } = obj); // ✅ Works fine
console.log(numbers); // [1, 2]
```

2. **When using destructuring in an arrow function body:**

```
const setValues = (obj) => ({ a: numbers[0], b: numbers[1] } = obj);
```

**🔥 When Don’t You Need Parentheses?**

When destructuring during **variable declaration** using let, const, or var:

```
const obj = { a: 1, b: 2 };
const { a: x, b: y } = obj; // ✅ No parentheses needed
console.log(x, y); // 1, 2
```


**🎯 Summary:**

  

Use parentheses when destructuring **outside of declarations** to avoid ambiguity with block statements:


✅ **Needs parentheses**:

• In assignment statements.

• In arrow functions.


✅ **No parentheses needed**:

• In variable declarations (with const, let, var).

</br>

---