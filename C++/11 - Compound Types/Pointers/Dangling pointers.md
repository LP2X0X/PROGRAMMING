- Much like a [[Dangling references|dangling reference]], a **dangling pointer** is a pointer that is holding the address of an object that is no longer valid (e.g. because it has been destroyed).
- Dereferencing a dangling pointer (e.g. in order to print the value being pointed at) will lead to undefined behavior, as you are trying to access an object that is no longer valid.

```ad-note
Dereferencing an invalid pointer will lead to undefined behavior. Any other use of an invalid pointer value is implementation-defined.
```