- A **copy constructor** is a constructor that is used to initialize an object with an existing object of the same type. After the copy constructor executes, the newly created object should be a copy of the object passed in as the initializer.

```ad-note
If you do not provide a copy constructor for your classes, C++ will create a public **implicit copy constructor** for you.
```

- Copy constructors should have no side effects beyond copying. This is because the compiler may optimize the copy constructor out in certain cases. If you are relying on the copy constructor for some behavior other than just copying, that behavior may or may not occur.

```ad-note
It is a requirement that the parameter of a copy constructor be an lvalue reference or const lvalue reference. Because the copy constructor should not be modifying the parameter, using a const lvalue reference is preferred.
```

```ad-important
When an object is passed by value, the argument is copied into the parameter. When the argument and parameter are the same class type, the copy is made by implicitly invoking the copy constructor.
When the return type and the return value are the same class type, the temporary object is initialized by implicitly invoking the copy constructor.
```

- If we prefer, we can explicitly request the compiler create a default copy constructor for us using the `= default` syntax:

```cpp
    // Explicitly request default copy constructor
    Fraction(const Fraction& fraction) = default;
```

- Occasionally we run into cases where we do not want objects of a certain class to be copyable. We can prevent this by marking the copy constructor function as deleted, using the `= delete` syntax:

```cpp
    // Delete the copy constructor so no copies can be made
    Fraction(const Fraction& fraction) = delete;
```