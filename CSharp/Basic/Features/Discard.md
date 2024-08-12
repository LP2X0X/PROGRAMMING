In C#, the underscore symbol `_` can be used as a discard symbol in several contexts to indicate that a value is being intentionally ignored or discarded.
Here are some common use cases for the discard symbol `_`:

1. **Unused Variables**: When you declare variables that you don't intend to use, you can use `_` to indicate that the value is being ignored.

    ```csharp
    var _ = SomeMethodThatReturnsValue();
    ```

2. **Unused Parameters**: In method signatures or lambda expressions, you can use `_` to represent parameters that are not used within the method body.

    ```csharp
    void SomeMethod(int x, int _)
    {
        // Use only 'x', discard '_'
    }

    _ = SomeMethodThatReturnsValue();
    ```

3. **Discarding Tuple Elements**: When deconstructing tuples or other types that support deconstruction, you can use `_` to discard elements that you don't need.

    ```csharp
    (int x, _, int z) = GetCoordinates();
    ```

4. **Ignored Out Parameters**: When calling methods with `out` parameters but you don't need to use all of them, you can use `_` to indicate that the value is being discarded.

    ```csharp
    SomeMethodWithOutParameter(out _, out var y);
    ```

Using `_` as a discard symbol can improve code readability and clearly communicate the intention to ignore certain values.