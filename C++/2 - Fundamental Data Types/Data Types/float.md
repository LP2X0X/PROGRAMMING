- A **floating point** type variable is a variable that can hold a number with a fractional component, such as 4320.0, -3.33, or 0.01226. The _floating_ part of the name _floating point_ refers to the fact that the decimal point can “float” -- that is, it can support a variable number of digits before and after the decimal point.

| Category       | Type        | Typical Size       |
| -------------- | ----------- | ------------------ |
| floating point | float       | 4 bytes            |
|                | double      | 8 bytes            |
|                | long double | 8, 12, or 16 bytes |


| Size                                    | Range                                                           | Precision                              |
| --------------------------------------- | --------------------------------------------------------------- | -------------------------------------- |
| 4 bytes                                 | ±1.18 x 10<sup>-38</sup> to ±3.4 x 10<sup>38</sup> and 0.0      | 6-9 significant digits, typically 7    |
| 8 bytes                                 | ±2.23 x 10<sup>-308</sup> to ±1.80 x 10<sup>308</sup> and 0.0   | 15-18 significant digits, typically 16 |
| 80-bits (typically uses 12 or 16 bytes) | ±3.36 x 10<sup>-4932</sup> to ±1.18 x 10<sup>4932</sup> and 0.0 | 18-21 significant digits               |
| 16 bytes                                | ±3.36 x 10<sup>-4932</sup> to ±1.18 x 10<sup>4932</sup> and 0.0 | 33-36 significant digits               |

- Reference: https://fabiensanglard.net/floating_point_visually_explained/index.html