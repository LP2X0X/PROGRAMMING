- The best way to tell the compiler to look in a particular namespace for an identifier is to use the **scope resolution operator** (::). The scope resolution operator tells the compiler that the identifier specified by the right-hand operand should be looked for in the scope of the left-hand operand.
```ad-note
The scope resolution operator can also be used in front of an identifier without providing a namespace name (e.g. `::doSomething`). In such a case, the identifier (e.g. `doSomething`) is looked for in the global namespace.
```

```ad-note
It’s legal to declare namespace blocks in multiple locations (either across multiple files, or multiple places within the same file). All declarations within the namespace are considered part of the namespace.
```