---
tags:
  - cpp
  - term
  - fundamental
---
```cpp
int width = 5; // copy-initialization of value 5 into variable width
```

- When an initial value is provided after an equals sign, this is called **copy-initialization**. This form of initialization was inherited from the C language.
- It **copies** the value on the right-hand side of the equals into the variable being created on the left-hand side.
- May involve creating a temporary and then copying or moving it into the variable on the left-hand side.

```ad-note
Copy-initialization is also used whenever values are **implicitly copied**, such as when *passing arguments to a function by value*, *returning from a function by value*, or catching exceptions by value.
```

---

## ❓Why does copy-initialization potentially create a temporary, instead of just constructing directly like direct-initialization?

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

---

Great question — the reason **copy initialization does not use `explicit` constructors** is deeply rooted in **how C++ was designed to balance safety and convenience**.

Let’s walk through the logic:

---

## ❓ Why doesn’t copy initialization allow `explicit` constructors?

### 🔑 Core idea:

> **Copy initialization** allows **implicit conversions**.  
> **`explicit` constructors are meant to block implicit conversions.**

So if a constructor is marked `explicit`, it's the programmer saying:

> “Don’t let the compiler use this constructor unless I call it **directly and intentionally**.”

---

### 🔍 Copy Initialization Means Implicit Conversion

Example:

```cpp
struct A {
    explicit A(int x) {}
};

A a1(10);      // ✅ Direct init — allowed
A a2 = 10;     // ❌ Copy init — error: explicit constructor not allowed
```

Why does `A a2 = 10;` fail?

Because:

1. The compiler sees `10`, sees that `A` can be constructed from `int`,
    
2. And **wants to convert** the `int` into an `A` object implicitly.
    
3. But `explicit A(int)` says: **No implicit conversions!**
    

So the compiler **rejects it**.

> **Copy initialization is intentionally restricted from using `explicit` constructors to prevent unintended conversions.**  

---

```ad-important
The implicit conversion in copy-initialization must produce T directly from the initializer, while, e.g. direct-initialization expects an implicit conversion from the initializer to an argument of T's constructor.
```

---

## 🔍 What This Means — Explained Simply

### 🚗 Copy Initialization:

> Converts the initializer **directly into the target type `T`**.

```cpp
T obj = initializer;
```

- The compiler says: “Can I convert `initializer` directly into a `T`?”
    
- It looks for a **single-step** conversion that results in a `T` object.
    
- It **cannot** chain through intermediate types or constructors marked `explicit`.
    

### 🎯 Direct Initialization:

> Tries to find a **constructor of `T`** that accepts the initializer — **even if that means converting the initializer first**.

```cpp
T obj(initializer);  // or T obj{initializer};
```

- The compiler says: “Do any of `T`'s constructors take something that looks like this initializer?”
    
- It allows **implicit conversion of the initializer to the constructor parameter**.
    

---

## ✨ Example to Illustrate the Difference

```cpp
struct B {
    B(int) {}
};

struct A {
    explicit A(B) {}
};
```

### ✅ Direct Initialization

```cpp
A a(B(42));       // OK: B(42) constructs a B, passed directly to A(B)
A a2 = A(B(42));  // OK: temporary A created, then moved/copied into a2
```

### ❌ Copy Initialization

```cpp
A a3 = B(42);     // ❌ Error: copy init not allowed with `explicit A(B)`
```

Why?

- **Copy init** asks: “Can I make an `A` directly from `B(42)`?”
    
    - This requires the compiler to **implicitly convert** `B(42)` to an `A`, but that constructor is `explicit`, so it’s blocked.
        
- **Direct init** just calls `A(B(42))`, which is allowed — you're being explicit.
    

---

## 🔑 Summary

|Aspect|Copy Initialization|Direct Initialization|
|---|---|---|
|Implicit conversion|Must produce `T` directly|Converts to constructor argument|
|`explicit` constructors|❌ Ignored|✅ Allowed|
|Flexibility|Limited (safe)|More flexible|
