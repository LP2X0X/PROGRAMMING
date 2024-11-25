- Besides a memory address, there is one additional value that a pointer can hold: a null value. A **null value** (often shortened to **null**) is a special value that means something has no value. When a pointer is holding a null value, it means the pointer is not pointing at anything. Such a pointer is called a **null pointer**.
- Much like the keywords `true` and `false` represent Boolean literal values, the **nullptr** keyword represents a null pointer literal. We can use `nullptr` to explicitly initialize or assign a pointer a null value.
- Much like dereferencing a dangling (or wild) pointer leads to undefined behavior, dereferencing a null pointer also leads to undefined behavior. In most cases, it will crash your application.

```ad-note
Conditionals can only be used to differentiate null pointers from non-null pointers. There is no convenient way to determine whether a non-null pointer is pointing to a valid object or dangling (pointing to an invalid object).
```