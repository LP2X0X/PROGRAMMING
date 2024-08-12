## Tuples vs `System.Tuple`

C# tuples, which are backed by [System.ValueTuple](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple) types, are different from tuples that are represented by [System.Tuple](https://learn.microsoft.com/en-us/dotnet/api/system.tuple) types. The main differences are as follows:

- `System.ValueTuple` types are [value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types). `System.Tuple` types are [reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types).
- `System.ValueTuple` types are mutable. `System.Tuple` types are immutable.
- Data members of `System.ValueTuple` types are fields. Data members of `System.Tuple` types are properties.