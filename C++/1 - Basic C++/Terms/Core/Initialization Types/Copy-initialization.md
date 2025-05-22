---
tags: cpp, term, fundamental
---
```cpp
int width = 5; // copy-initialization of value 5 into variable width
```

- When an initial value is provided after an equals sign, this is called **copy-initialization**. This form of initialization was inherited from the C language.
- It copies the value on the right-hand side of the equals into the variable being created on the left-hand side.
- May involve creating a temporary and then copying or moving it into the variable on the left-hand side.

```ad-note
Copy-initialization is also used whenever values are **implicitly copied**, such as when passing arguments to a function by value, returning from a function by value, or catching exceptions by value.
```

---

## Why does copy-initialization potentially create a temporary, instead of just constructing directly like direct-initialization?

---

### 🧠 Historical & Design Reason: **Consistency with Assignment**

#### Think of this:

```cpp
MyClass obj = some_value;
```

This looks a lot like **assignment**, right?

The C++ committee wanted copy-initialization to work **like assignment** — so when you write:

```cpp
MyClass obj = 42;
```

…it behaves like:

```cpp
MyClass temp = MyClass(42);  // create a temporary
MyClass obj = temp;          // copy or move constructor
```

- This gave the language a **uniform model**:
	- `=` means "take a value and assign it"
	- `T obj = value;` is conceptually treated as:
	    - Convert `value` to a `T`
	    - Then use that to initialize `obj` (via copy/move)

---

### 📋 Example: Implicit Conversion

```cpp
class MyClass {
public:
    MyClass(int) { };  // converting constructor
};

MyClass obj = 10;  // copy-initialization
```

Here, `10` is **implicitly converted** to a `MyClass` object. So:
- The compiler creates a temporary `MyClass(10)`
- Then uses that to copy-construct `obj`

➡️ This flexibility is why **copy-initialization is more permissive** (e.g., allows implicit conversions) — at the cost of a conceptual temporary.

---

## 🧪 Why Not Always Optimize It?

- Even though compilers **now eliminate the temporary** (with **Copy Elision** and **RVO**), the **language rules** still treat them as separate steps to allow for:
	- Better overload resolution logic
	- Checking constructor eligibility
	- Supporting implicit conversions

So the temporary is part of the **semantics**, even if it's optimized away.

---

## 🔑 Summary: Why copy-initialization creates a temporary

> **Because it's meant to behave like assignment.** The language model is:
> 
> 1. Convert the right-hand side into a temporary of type `T`
>     
> 2. Use that temporary to initialize the object
>     

This separation allows **implicit conversions**, aligns with **assignment semantics**, and supports **legacy C++ idioms**.
