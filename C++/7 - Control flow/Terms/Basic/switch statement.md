- Because testing a variable or expression for equality against a set of different values is common, C++ provides an alternative conditional statement called a **switch statement** that is specialized for this purpose.
- The idea behind a **switch statement** is simple: an expression (sometimes called the `condition`) is evaluated to produce a value. If the expression’s value is equal to the value after any of the `case labels`, the statements after the matching [[case label]] are executed. If no matching value can be found and a [[default label]] exists, the statements after the `default label` are executed instead.

```ad-important
The one restriction is that the condition must evaluate to an integral type or an enumerated type, or be convertible to one.
```

```ad-note
With switch statements, the statements after labels are all scoped to the switch block. No [[Implicit blocks|implicit blocks]] are created.
```

```ad-tip
Best practice:
If defining variables used in a case statement, do so in a block inside the case.
```