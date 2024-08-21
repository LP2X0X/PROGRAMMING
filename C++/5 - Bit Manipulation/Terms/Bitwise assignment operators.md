|Operator|Symbol|Form|Operation|
|---|---|---|---|
|Left shift assignment|<<=|x <<= y|Shift x left by y bits|
|Right shift assignment|>>=|x >>= y|Shift x right by y bits|
|Bitwise OR assignment|\|=|x \|= y|Assign x \| y to x|
|Bitwise AND assignment|&=|x &= y|Assign x & y to x|
|Bitwise XOR assignment|^=|x ^= y|Assign x ^ y to x|

---

```ad-warning
Bitwise operators will promote operands with narrower integral types to `int` or `unsigned int`.
`operator~` and `operator<<` are width-sensitive and may produce different results depending on the width of the operand.
`static_cast` the result of such bitwise operations back to the narrower integral type before using to ensure correct results.
```