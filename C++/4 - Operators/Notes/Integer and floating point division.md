---
tags:
  - cpp
  - note
---

- It is easiest to think of the [[Arithmetic operators#^2747fe|division operator]] as having two different “modes”.

- If either (or both) of the operands are floating point values, the _division operator_ performs floating point division. **Floating point division** returns a floating point value, and the fraction is kept. For example, `7.0 / 4 = 1.75`, `7 / 4.0 = 1.75`, and `7.0 / 4.0 = 1.75`. As with all floating point arithmetic operations, rounding errors may occur.

- If both of the operands are integers, the _division operator_ performs integer division instead. **Integer division** drops any fractions and returns an integer value. For example, `7 / 4 = 1` because the fractional portion of the result is dropped. Similarly, `-7 / 4 = -1` because the fraction is dropped.