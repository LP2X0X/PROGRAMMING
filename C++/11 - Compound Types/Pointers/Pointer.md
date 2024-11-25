### Definition
- A **pointer** is an object that holds a _memory address_ (typically of another variable) as its value. This allows us to store the address of some other object to use later.
- To create a pointer variable, we simply define a variable with a pointer type:
```cpp
int main()
{
    int x { 5 };    // normal variable
    int& ref { x }; // a reference to an integer (bound to x)

    int* ptr;       // a pointer to an integer

    return 0;
}
```

---

### Pointer initialization

- Like normal variables, pointers are _not_ initialized by default. A pointer that has not been initialized is sometimes called a **wild pointer**. Wild pointers contain a garbage address, and dereferencing a wild pointer will result in undefined behavior. Because of this, you should always initialize your pointers to a known value.

---

- When we use a pointer without a dereference (`ptr`), we are accessing the address held by the pointer. Modifying this (`ptr = &y`) changes what the pointer is pointing at.
- When we dereference a pointer (`*ptr`), we are accessing the object being pointed at. Modifying this (`*ptr = 6;`) changes the value of the object being pointed at.
- The size of a pointer is dependent upon the architecture the executable is compiled for regardless of the size of the object being pointed to.

```ad-note
The size of the pointer is always the same. This is because a pointer is just a memory address, and the number of bits needed to access a memory address is constant.
```