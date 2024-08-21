- **Relational operators** are operators that let you compare two values. There are 6 relational operators:

| Operator               | Symbol | Form   | Operation                                                |
| ---------------------- | ------ | ------ | -------------------------------------------------------- |
| Greater than           | >      | x > y  | true if x is greater than y, false otherwise             |
| Less than              | <      | x < y  | true if x is less than y, false otherwise                |
| Greater than or equals | >=     | x >= y | true if x is greater than or equal to y, false otherwise |
| Less than or equals    | <=     | x <= y | true if x is less than or equal to y, false otherwise    |
| Equality               | ==     | x == y | true if x equals y, false otherwise                      |
| Inequality             | !=     | x != y | true if x does not equal y, false otherwise              |

```ad-warning
Avoid using operator== and operator!= to compare floating point values if there is any chance those values have been calculated.
```