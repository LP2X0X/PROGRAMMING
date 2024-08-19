| Operator    | Symbol | Form      | Meaning                                                                |
| ----------- | ------ | --------- | ---------------------------------------------------------------------- |
| Conditional | ?:     | c ? x : y | If conditional `c` is `true` then evaluate `x`, otherwise evaluate `y` |

```ad-tip
Best practice
Parenthesize the entire conditional operator when used in a compound expression.
For readability, consider parenthesizing the condition if it contains any operators (other than the function call operator).
```

```ad-note
To comply with C++’s type checking rules, one of the following must be true:
- The type of the second and third operand must match.
- The compiler must be able to find a way to convert one or both of the second and third operands to matching types. The conversion rules the compiler uses are fairly complex and may yield surprising results in some cases.
```