```ad-warning
This is different from [[Pointer to const value|pointer to const value]].
```
- A **const pointer** is a pointer whose address can not be changed after initialization.
- To declare a const pointer, use the `const` keyword **after the asterisk** in the pointer declaration:

```cpp
int main()
{
    int x{ 5 };
    int* const ptr { &x }; // const after the asterisk means this is a const pointer

    return 0;
}
```

- Just like a normal const variable, a const pointer must be initialized upon definition, and this **value can’t be changed via assignment**:

```cpp
int main()
{
    int x{ 5 };
    int y{ 6 };

    int* const ptr { &x }; // okay: the const pointer is initialized to the address of x
    ptr = &y; // error: once initialized, a const pointer can not be changed.

    return 0;
}
```

- However, because the _value_ being pointed to is non-const, it is possible to change the value being pointed to via dereferencing the const pointer:

```cpp
int main()
{
    int x{ 5 };
    int* const ptr { &x }; // ptr will always point to x

    *ptr = 6; // okay: the value being pointed to is non-const

    return 0;
}
```