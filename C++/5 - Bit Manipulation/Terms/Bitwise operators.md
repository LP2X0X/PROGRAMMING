| Operator    | Symbol | Form   | Operation                          |
| ----------- | ------ | ------ | ---------------------------------- |
| left shift  | <<     | x << y | all bits in x shifted left y bits  |
| right shift | >>     | x >> y | all bits in x shifted right y bits |
| bitwise NOT | ~      | ~x     | all bits in x flipped              |
| bitwise AND | &      | x & y  | each bit in x AND each bit in y    |
| bitwise OR  | \|     | x \| y | each bit in x OR each bit in y     |
| bitwise XOR | ^      | x ^ y  | each bit in x XOR each bit in y    |

```ad-tip
Best practice: To avoid surprises, use the bitwise operators with unsigned operands or std::bitset.
```
---
