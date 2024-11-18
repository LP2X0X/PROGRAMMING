#### <u>Step 1</u>: The compiler tries to find an exact match.
- This happens in two phases. First, the compiler will see if there is an overloaded function where the type of the arguments in the function call exactly matches the type of the parameters in the overloaded functions.
- Second, the compiler will apply a number of trivial conversions to the arguments in the function call. The **trivial conversions** are a set of specific conversion rules that will modify types (without modifying the value) for purposes of finding a match. These include:
	- lvalue to rvalue conversions
	- Qualification conversions (e.g. non-const to const)
	- Non-reference to reference conversions

#### <u>Step 2</u>: If no exact match is found, the compiler tries to find a match by applying numeric promotion to the argument(s).

#### <u>Step 3</u>: If no match is found via numeric promotion, the compiler tries to find a match by applying numeric conversions to the arguments.

#### <u>Step 4</u>: If no match is found via numeric conversion, the compiler tries to find a match through any user-defined conversions.
```cpp
class X // this defines a new type called X
{
public:
    operator int() { return 0; } // Here's a user-defined conversion from X to int
};

void foo(int)
{
}

void foo(double)
{
}

int main()
{
    X x; // Here, we're creating an object of type X (named x)
    foo(x); // x is converted to type int using the user-defined conversion from X to int

    return 0;
}
```

#### <u>Step 5</u>: If no match is found via user-defined conversion, the compiler will look for a matching function that uses ellipsis.

#### <u>Step 6</u>: If no matches have been found by this point, the compiler gives up and will issue a compile error about not being able to find a matching function.

---

## Resolving ambiguous matches
- There are a few ways to resolve ambiguous matches:
	1. Often, the best way is simply to define a new overloaded function that takes parameters of exactly the type you are trying to call the function with. Then C++ will be able to find an exact match for the function call.
	2. Alternatively, explicitly cast the ambiguous argument(s) to match the type of the function you want to call.