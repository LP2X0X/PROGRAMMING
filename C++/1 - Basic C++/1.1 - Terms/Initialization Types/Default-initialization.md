---
tags:
  - cpp
  - term
  - fundamental
---
```cpp
int x;
```
- When no [[Initializer|initializer]] is provided, it is called default-initialization.

---
### 🔹 **Built-in Types (e.g., `int`, `float`)**

- When a built-in type is default-initialized without an explicit initializer:
	 - **Local (automatic) variables**: The variable is not initialized; it holds an indeterminate value.
	 - **Static or thread-local variables**: The variable is zero-initialized.

|Scope|Result|
|---|---|
|Static/global|✅ **Zero** initialized|
|Local (stack)|❌ **Garbage** (uninitialized)|

```cpp
int c; // ✅ Initialized to 0

int main() {
    int a;           // 🟥 Uninitialized — contains garbage!
    static int b;    // ✅ Initialized to 0
}
```

---

### 🔹 **Class Types**

- For class types, default-initialization invokes the default constructor.
- However, if the class has members of built-in types and the constructor does not explicitly initialize them, those members remain uninitialized (garbage).

```cpp
struct MyClass {
    int x;
    MyClass() { }  // does NOT initialize x
};

MyClass obj;  // default-initialized
```

---

### 🔹 **Arrays**

- When an array is default-initialized:
	- **Arrays of built-in types**: Each element is default-initialized individually, following the rules for built-in types.
	- **Arrays of class types**: Each element is default-initialized, invoking the default constructor for each object.

---

