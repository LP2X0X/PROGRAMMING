- A **default argument** is a default value provided for a function parameter. When making a function call, the caller can optionally provide an argument for any function parameter that has a default argument. If the caller provides an argument, the value of the argument in the function call is used. If the caller does not provide an argument, the value of the default argument is used.
- If a parameter is given a default argument, **all subsequent parameters (to the right) must also be given default arguments**.
- Functions with default arguments may be overloaded.
````ad-note
Note that you must use the equals sign to specify a default argument. Using parenthesis or brace initialization won’t work:
```cpp
void foo(int x = 5);   // ok
void goo(int x ( 5 )); // compile error
void boo(int x { 5 }); // compile error
````

```ad-note
If the function has a forward declaration (especially one in a header file), put the default argument there (we don't have to re define it in the definition). Otherwise, put the default argument in the function definition.
```

````ad-note
Default arguments can easily lead to ambiguous function calls:
```cpp
void foo(int x = 5);   // ok
void goo(int x ( 5 )); // compile error
void boo(int x { 5 }); // compile error
```cpp
void foo(int x = 0)
{
}

void foo(double d = 0.0)
{
}

int main()
{
    foo(); // ambiguous function call

    return 0;
}
````

```ad-tip
Default arguments are also useful in cases where we need to add a new parameter to an existing function.
```