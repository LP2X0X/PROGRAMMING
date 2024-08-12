- The final point of interest regarding .NET metadata is the fact that every string literal in your code base is documented under the User Strings token.

```ad-important
As illustrated in the previous metadata listing, always be aware that all strings are clearly documented in the assembly metadata. This could have huge security consequences if you were to use string literals to represent passwords, credit card numbers, or other sensitive information.
```