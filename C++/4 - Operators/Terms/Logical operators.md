| Operator    | Symbol | Example Usage | Operation                                                 |
| ----------- | ------ | ------------- | --------------------------------------------------------- |
| Logical NOT | !      | !x            | true if x is false, or false if x is true                 |
| Logical AND | &&     | x && y        | true if x and y are both true, false otherwise            |
| Logical OR  | \|     | x \| y        | true if either (or both) x or y are true, false otherwise |

---

| Logical NOT (operator !) |        |
| ------------------------ | ------ |
| Operand                  | Result |
| true                     | false  |
| false                    | true   |

---

- [De Morgan’s laws](https://en.wikipedia.org/wiki/De_Morgan%27s_laws) tell us how the _logical NOT_ should be distributed in these cases:
`!(x && y)` is equivalent to `!x || !y`  
`!(x || y)` is equivalent to `!x && !y`

---
### Alternative operator representations
- Many operators in C++ (such as operator ||) have names that are just symbols. Historically, not all keyboards and language standards have supported all of the symbols needed to type these operators. As such, C++ supports an alternative set of keywords for the operators that use words instead of symbols. For example, instead of `||`, you can use the keyword `or`.
- The full list can be found [here](https://en.cppreference.com/w/cpp/language/operator_alternative). Of particular note are the following three:

|Operator name|Keyword alternate name|
|---|---|
|&&|and|
|\||or|
|!|not|

- This means the following are identical:
```cpp
std::cout << !a && (b || c);
std::cout << not a and (b or c);
```

- While these alternative names might seem easier to understand right now, most experienced C++ developers prefer using the symbolic names over the keyword names. As such, we recommend learning and using the symbolic names, as this is what you will commonly find in existing code.