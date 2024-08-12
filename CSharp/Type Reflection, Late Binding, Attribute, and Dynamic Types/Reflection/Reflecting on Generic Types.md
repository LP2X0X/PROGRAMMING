- When you call Type.GetType() to obtain metadata descriptions of generic types, you must make use of a special syntax involving a “backtick” character (\`) followed by a numerical value that represents the number of type parameters the type supports. For example, if you want to print out the metadata description of System.Collections.Generic.List\<T>, you need to pass the following string into your application:
```csharp
System.Collections.Generic.List`1
```
- Here, you are using the numerical value of 1, given that List \<T> has only one type parameter. However, if you want to reflect over Dictionary\<TKey, TValue>, supply the value 2, like so:
```csharp
System.Collections.Generic.Dictionary`2
```