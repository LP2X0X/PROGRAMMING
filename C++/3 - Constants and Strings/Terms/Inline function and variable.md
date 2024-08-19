### Inline expansion
- **Inline expansion** is a process **by the compiler** where a function call is replaced by the code from the called function’s definition. This allows us to avoid the overhead of those calls, while preserving the results of the code.
- [[Performance overhead of function called|Beyond removing the cost of function call]], inline expansion can also allow the compiler to optimize the resulting code more efficiently -- for example, because the expression `((5 < 6) ? 5 : 6)` is now a constant expression, the compiler could further optimize the first statement in `main()` to `std::cout << 5 << '\n';`.

### The inline keyword, modernly
- In previous chapters, we mentioned that you should not implement functions (with external linkage) in header files, because when those headers are included into multiple .cpp files, the function definition will be copied into multiple .cpp files. These files will then be compiled, and the linker will throw an error because it will note that you’ve defined the same function more than once, which is a violation of the one-definition rule.
- In modern C++, the term `inline` has evolved to mean “multiple definitions are allowed”. Thus, an inline function is one that is allowed to be defined in multiple translation units (without violating the ODR).
- Inline functions have two primary requirements:
	- The compiler needs to be able to see the full definition of an inline function in each translation unit where the function is used (a forward declaration will not suffice on its own). The definition can occur after the point of use if a forward declaration is also provided. Only one such definition can occur per translation unit, otherwise a compilation error will occur.
	- Every definition for an inline function must be identical, otherwise undefined behavior will result.
```ad-note
The linker will consolidate all inline function definitions for an identifier into a single definition (thus still meeting the requirements of the one definition rule).
```

```ad-tip
Inline functions are typically defined in header files, where they can be \#included into the top of any code file that needs to see the full definition of the identifier.
```