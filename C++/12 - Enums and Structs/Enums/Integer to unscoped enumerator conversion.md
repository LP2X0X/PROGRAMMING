1. You can explicitly convert an integer to an unscoped enumerator using `static_cast`:

```cpp
enum Pet // no specified base
{
    cat, // assigned 0
    dog, // assigned 1
    pig, // assigned 2
    whale, // assigned 3
};

int main()
{
    Pet pet { static_cast<Pet>(2) }; // convert integer 2 to a Pet
    pet = static_cast<Pet>(3);       // our pig evolved into a whale!

    return 0;
}
```
- It is also safe to static_cast any integral value that is in range of the target enumeration’s underlying type, even if there are no enumerators representing that value. Static casting a value outside the range of the underlying type will result in undefined behavior.

2. As of C++17, if an unscoped enumeration has an explicitly specified base, then the compiler will allow you to list initialize an unscoped enumeration using an integral value:
```cpp
enum Pet: int // we've specified a base
{
    cat, // assigned 0
    dog, // assigned 1
    pig, // assigned 2
    whale, // assigned 3
};

int main()
{
    Pet pet1 { 2 }; // ok: can brace initialize unscoped enumeration with specified base with integer (C++17)

    return 0;
}
```