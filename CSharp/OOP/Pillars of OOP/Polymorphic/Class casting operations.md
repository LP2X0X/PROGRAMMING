- The first law of casting between class types is that when two classes are related by an “is-a” relationship, it is always safe to store a derived object within a base class reference. Formally, this is called an implicit cast, as “it just works” given the laws of inheritance.
- This is the second law of casting: you can, in such cases, explicitly downcast using the C# casting operator.
```ad-warning
Be aware that explicit casting is evaluated at runtime, not compile time.
```