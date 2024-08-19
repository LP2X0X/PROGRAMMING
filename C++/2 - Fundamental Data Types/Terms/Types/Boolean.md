- Boolean variables are variables that can have only two possible values: `true`, and `false`.
- If you want `std::cout` to print `true` or `false` instead of `0` or `1`, you can output `std::boolalpha`. This doesn’t output anything, but manipulates the way `std::cout` outputs bool values.
- When using <u>uniform initialization</u>, you can initialize a variable using integer literals `0` (for `false`) and `1` (for `true`) (but you really should be using `false` and `true` instead). Other integer literals cause compilation errors!

```ad-note
Copy initialization allows implicit conversion from int to bool
```

```ad-info
To allow `std::cin` to accept `false` and `true` as inputs, you must first input to `std::boolalpha`. However, when `std::boolalpha` is enabled for input, `0` and `1` will no longer be interpreted as Booleans inputs.
```

```ad-tip
Boolean values are often used as the return values for functions that check whether something is true or not. Such functions are typically named starting with the word _is_ (e.g. isEqual) or _has_ (e.g. hasCommonDivisor).
```