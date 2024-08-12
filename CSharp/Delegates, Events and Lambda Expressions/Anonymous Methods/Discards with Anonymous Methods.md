```ad-note
Because the underscore (_) was a legal variable identifier in previous versions of C#, there must be two or more discards used with the anonymous method to be treated as discards.
```

```ad-example
````csharp
Console.WriteLine("******** Discards with Anonymous Methods ********");  

Func<int,int,int> constant = delegate (int _, int _) {return 42;};  
Console.WriteLine("constant(3,4)={0}",constant(3,4));
```