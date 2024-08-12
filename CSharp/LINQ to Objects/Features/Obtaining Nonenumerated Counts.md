Using `Count()` after a LINQ query can affect performance because it may require the entire sequence to be enumerated to determine the count. This can be particularly costly for large datasets or when working with deferred execution in LINQ, where the query is not executed until the result is enumerated.

### Performance Impact of `Count()`

1. **Deferred Execution**: Many LINQ methods use deferred execution, meaning they don't execute the query until the result is iterated over. Calling `Count()` forces the immediate execution of the query, which can be expensive if the underlying data source is large or complex.
   
2. **Enumeration**: If the source does not have a direct way to count elements efficiently, `Count()` will enumerate through all elements. For instance, with an `IEnumerable<T>`, it would iterate through the entire sequence.

3. **Complex Queries**: If the LINQ query involves multiple operations like filtering, projecting, or joining, computing the count involves executing all these operations, which can be computationally intensive.

### Role of `TryGetNonEnumeratedCount`

`TryGetNonEnumeratedCount` is a method introduced in .NET 6 to improve performance when you need to count elements in a sequence. It attempts to get the count of elements in a sequence without forcing a full enumeration.

Here’s how it works and why it’s useful:

1. **Optimization for Known Collections**: For collections that inherently know their count (like arrays, lists, or other collections implementing `ICollection<T>`), `TryGetNonEnumeratedCount` can retrieve the count directly. This avoids the need for full enumeration.

2. **Fallback Mechanism**: If the count cannot be determined without enumeration (e.g., for an `IEnumerable<T>` without a known count), `TryGetNonEnumeratedCount` will fail gracefully, allowing you to decide whether to proceed with enumeration or handle it differently.

### Example Usage

```csharp
public static int GetCount(IEnumerable<int> source)
{
    if (source.TryGetNonEnumeratedCount(out int count))
    {
        return count;  // Efficiently returns the count if available
    }

    // Fallback to full enumeration if count is not directly available
    return source.Count();
}
```

In this example, `TryGetNonEnumeratedCount` first checks if the count is readily available. If so, it returns the count without iterating through the sequence. If not, it falls back to using `Count()`, which would enumerate the sequence.

### Benefits

1. **Performance**: By avoiding full enumeration where possible, `TryGetNonEnumeratedCount` can significantly improve performance, especially for large collections.
   
2. **Flexibility**: It provides a flexible way to attempt an optimized count operation, with a fallback to the standard approach when necessary.

Using `TryGetNonEnumeratedCount` can help mitigate the performance impact of using `Count()` after a LINQ query, making your code more efficient in scenarios where counting is a frequent operation.

---

- Reference: https://steven-giesel.com/blogPost/c341ffbc-d816-49ca-82a7-0e0e3b3e5910