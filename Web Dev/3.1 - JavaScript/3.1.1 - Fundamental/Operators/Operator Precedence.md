---
tags:
  - js
  - summary
  - fundamental
---

Here's a clear summary of **JavaScript operator precedence**, from **highest (evaluated first)** to **lowest (evaluated last)**:

|Precedence|Operator Type|Operators|Associativity|
|---|---|---|---|
|1|**Member access / call**|`.` `[]` `()` `?.`|Left to right|
|2|**New** (w/ args)|`new MyFunc()`|Right to left|
|3|**Function call**|`()`|Left to right|
|4|**Postfix**|`++` `--`|Left to right|
|5|**Unary**|`+` `-` `!` `~` `++` `--` `typeof` `delete`|Right to left|
|6|**Exponentiation**|`**`|Right to left|
|7|**Multiplicative**|`*` `/` `%`|Left to right|
|8|**Additive**|`+` `-`|Left to right|
|9|**Bitwise shift**|`<<` `>>` `>>>`|Left to right|
|10|**Relational**|`<` `<=` `>` `>=` `in` `instanceof`|Left to right|
|11|**Equality**| =​=  !​=  =​=​=  !​=​= |Left to right|
|12|**Bitwise AND**|`&`|Left to right|
|13|**Bitwise XOR**|`^`|Left to right|
|14|**Bitwise OR**|`\|`|Left to right|
|15|**Logical AND**|`&&`|Left to right|
|16|**Logical OR**|`\|`|Left to right|
|17|**Nullish coalescing**|`??`|Right to left|
|18|**Conditional (ternary)**|`cond ? a : b`|Right to left|
|19|**Assignment**|`=` `+=` `-=` `*=` etc.|Right to left|
|20|**Comma**|`,`|Left to right|

---

### ⚠️ Note:

- `??` (nullish coalescing) **cannot be mixed directly** with `||` or `&&` without parentheses.
- Use **parentheses** `()` liberally to clarify intention and avoid surprises.
