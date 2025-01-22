- We can use the **explicit** keyword to tell the compiler that a constructor should not be used as a converting constructor.

```ad-important
Making a constructor `explicit` has two notable consequences:

- An explicit constructor cannot be used to do copy initialization or copy list initialization.
- An explicit constructor cannot be used to do implicit conversions (since this uses copy initialization or copy list initialization).
```

- Make any constructor that accepts a single argument `explicit` by default. If an implicit conversion between types is both semantically equivalent and performant, you can consider making the constructor non-explicit.
- Do not make copy or move constructors explicit, as these do not perform conversions.