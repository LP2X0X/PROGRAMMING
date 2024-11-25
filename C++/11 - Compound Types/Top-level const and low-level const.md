In C++, **const** can be applied at different levels, and understanding the distinction between **top-level const** and **low-level const** is crucial for clarity in complex type declarations, especially when dealing with pointers and references.

---

## **1. Top-Level `const`**

- Refers to the immutability of the **object itself**.
- Applies to the variable directly.
- Indicates that the object or variable cannot be modified.

### **Example**

```cpp
const int x = 10;  // x is top-level const
```

Here:

- `x` itself is constant and cannot be changed.
- Top-level const applies to `x`, not to its contents or type.

---

## **2. Low-Level `const`**

- Refers to the immutability of **what the variable points to or references**.
- Applies to the underlying object or the data being pointed to.

### **Example**

```cpp
const int* ptr = &x;  // ptr is a pointer to a const int
```

Here:

- The `int` being pointed to by `ptr` is constant (low-level const).
- You cannot modify the object through `ptr`.

---

## **Difference between Top-Level and Low-Level `const`**

|**Aspect**|**Top-Level `const`**|**Low-Level `const`**|
|---|---|---|
|**What it applies to**|The object or variable itself.|The object being pointed to or referenced.|
|**Modifiability**|Prevents modification of the object.|Prevents modification of the object through a pointer or reference.|
|**Examples**|`const int x = 10;`|`const int* ptr = &x;`|

---

## **3. Mixed Example**

```cpp
int x = 5;
const int y = 10;

const int* ptr1 = &x;  // Low-level const: data pointed to by ptr1 is const
int* const ptr2 = &x;  // Top-level const: ptr2 is const (cannot change address)
const int* const ptr3 = &y;  // Both: ptr3 is const, and the data it points to is const
```

- `const int* ptr1`:
    
    - Low-level const applies to `int`.
    - `*ptr1` (the value being pointed to) cannot be modified.
    - `ptr1` itself can point to another address.
- `int* const ptr2`:
    
    - Top-level const applies to `ptr2`.
    - `ptr2` cannot point to another address.
    - However, the value `*ptr2` can be modified.
- `const int* const ptr3`:
    
    - Both top-level and low-level const.
    - `ptr3` cannot point to another address.
    - The value `*ptr3` cannot be modified.

---

## **4. Const with Functions**

When using functions, top-level and low-level consts can appear in various contexts.

### **Pass by Pointer**

```cpp
void func(const int* ptr);  // Low-level const
void func(int* const ptr);  // Top-level const
void func(const int* const ptr);  // Both
```

### **Pass by Reference**

```cpp
void func(const int& ref);  // Low-level const: ref cannot modify the original value
```

---

## **5. Const and Type Qualifiers with `const_cast`**

You can use `const_cast` to remove low-level constness (not top-level constness). Modifying an object through a casted pointer to a `const` variable leads to **undefined behavior** if the object was originally declared as `const`.

### Example:

```cpp
const int x = 5;
int* ptr = const_cast<int*>(&x);
*ptr = 10;  // Undefined behavior, as x was originally const
```

---

## **6. Practical Usage**

- Use **top-level const** when you want to ensure a variable remains constant and cannot be reassigned.
- Use **low-level const** when you want to ensure that a pointer or reference doesn't modify the value it points to.

Understanding the distinction ensures that you write safer and more predictable code in scenarios involving const correctness.

---

#### Reference
https://www.linkedin.com/pulse/demystifying-top-level-vs-low-level-const-ali-el-bana-4dpof