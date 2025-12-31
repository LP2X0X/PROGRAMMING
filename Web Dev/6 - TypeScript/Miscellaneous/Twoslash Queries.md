---
tags: 
 - typescript
 - miscellaneous
---

In TypeScript, the `// ^?` notation is a **Twoslash query**. It is a special comment syntax used to show the **inferred type** of a variable or expression directly in your editor without needing to hover over it.

While this was originally created for the official TypeScript documentation and playground, it has become a popular tool for developers via VS Code extensions.

---

### 1. How it works

When you place `// ^?` on the line below a piece of code, the "hat" (`^`) points to the specific symbol you want to inspect. An extension then renders the type information as an **Inlay Hint** (ghost text) at the end of that comment.

**Example:**

```ts
const message = "Hello World";
//    ^? const message: "Hello World"

const numbers = [1, 2, 3].map(n => n.toString());
//    ^? const numbers: string[]
```

---

### 2. How to get it in VS Code

By default, VS Code treats `// ^?` as a regular comment and does nothing. To see the type information, you need to install an extension:

- **Extension Name:** `vscode-twoslash-queries` (by Orta Therox).
    
- **What it does:** It watches for that specific comment pattern and uses the TypeScript language service to display the type results inline.
    

---

### 3. Why use it?

- **"Keyboard Warrior" Workflow:** You don't have to reach for your mouse to hover over a variable to see its type.
    
- **Debugging Complex Types:** If you are building complex **Mapped Types** or deeply nested generics, you can leave these comments in your file to see exactly how the type is resolving as you make changes.
    
- **Code Documentation:** It is great for tutorials or team projects where you want to show the resulting type of a complex operation explicitly in the code.
    

---

### 4. Advanced "Twoslash" Notations

If you have the extension installed, there are a few other shorthand tricks:

| **Shortcut** | **Effect**                                                                         |
| ------------ | ---------------------------------------------------------------------------------- |
| `// ^?`      | Shows the type of the symbol above the caret (`^`).                                |
| `//=>`       | (At the end of a line) Shows the type of the leftmost named variable on that line. | 
| `// @errors` | In the TS Playground, this tells the compiler to show all errors in the block.     |

---

### Tips for use

- **Alignment matters:** Make sure the `^` symbol is directly underneath the variable or property you want to check.
    
- **Settings:** You can customize how these look in VS Code by searching for `twoslash` in your settings after installing the extension (e.g., changing the color or maximum length of the hint).
    