- When we define a struct (or class) type, we can provide a default initialization value for each member as part of the type definition. For members not marked as `static`, this process is sometimes called **non-static member initialization**. The initialization value is called a **default member initializer**.

```cpp
struct Something
{
    int x;       // no initialization value (bad)
    int y {};    // value-initialized by default
    int z { 2 }; // explicit default value
};

int main()
{
    Something s1; // s1.x is uninitialized, s1.y is 0, and s1.z is 2

    return 0;
}
```