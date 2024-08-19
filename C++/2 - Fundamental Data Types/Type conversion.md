- The process of converting a value from one type to another type is called **type conversion**.
- When the compiler does type conversion on our behalf without us explicitly asking, we call this **implicit type conversion**.

---

### Explicit type conversion
- **Explicit type conversion** allow us (the programmer) to explicitly tell the compiler to convert a value from one type to another type, and that we take full responsibility for the result of that conversion. If such a conversion results in the loss of value, the compiler will not warn us.
- To perform an explicit type conversion, in most cases we’ll use the `static_cast` operator. The syntax for the `static cast` looks a little funny:
```cpp
static_cast<new_type>(expression)
```

```ad-note
It’s worth noting that the argument to _static_cast_ evaluates as an expression. When we pass in a variable, that variable is evaluated to produce its value, and that value is then converted to the new type. The variable itself is _not_ affected by casting its value to a new type.
```

```ad-important
std::int8_t and std::uint8_t likely behave like chars instead of integers
```