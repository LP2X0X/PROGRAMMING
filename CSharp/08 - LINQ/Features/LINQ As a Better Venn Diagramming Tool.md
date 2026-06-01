---
tags:
 - csharp
 - linq
---

LINQ (Language Integrated Query) can indeed be seen as a powerful tool for performing operations analogous to [[Venn diagram]], which represent the relationships between sets. In LINQ, various set operations like union, intersection, difference, and complement can be performed efficiently on collections, making it an excellent tool for these purposes. Below are some key operations and examples demonstrating how LINQ can serve as a better Venn diagramming tool.

### LINQ Set Operations

1. **Union**: Combines two sets to include all distinct elements from both sets.
2. **Intersection**: Returns only the elements that are present in both sets.
3. **Difference**: Returns elements that are in the first set but not in the second set.
4. **Complement**: Returns elements that are in the universal set but not in the given set.
5. **Concat**: Concatenates two sequences, appending the second sequence to the first.
6. **Distinct**: Removes duplicate elements from a single set.
7. **Symmetric Difference**: Returns elements that are in either set but not in both.
8. **UnionBy / IntersectBy / ExceptBy / DistinctBy** (.NET 6+): Perform set operations using a key selector instead of comparing entire elements.

### LINQ Operations Examples

#### 1. Union

Combines elements from two collections, removing duplicates.

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var union = setA.Union(setB);

foreach (var item in union)
{
    Console.WriteLine(item); // Output: 1, 2, 3, 4, 5
}
```

#### 2. Intersection

Finds common elements between two collections.

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var intersection = setA.Intersect(setB);

foreach (var item in intersection)
{
    Console.WriteLine(item); // Output: 3
}
```

#### 3. Difference (Except)

Finds elements in one collection that are not in the other.

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var difference = setA.Except(setB);

foreach (var item in difference)
{
    Console.WriteLine(item); // Output: 1, 2
}
```

#### 4. Complement

To find the complement of a set (elements in the universal set but not in the given set), you need to define the universal set and then subtract the given set from it.

```csharp
var universalSet = new List<int> { 1, 2, 3, 4, 5, 6 };
var setA = new List<int> { 1, 2, 3 };

var complement = universalSet.Except(setA);

foreach (var item in complement)
{
    Console.WriteLine(item); // Output: 4, 5, 6
}
```

#### 5. Concat

Concatenates two sequences, appending the second sequence to the first without removing duplicates.

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var concatenated = setA.Concat(setB);

foreach (var item in concatenated)
{
    Console.WriteLine(item); // Output: 1, 2, 3, 3, 4, 5
}
```

#### 6. Distinct

Removes duplicate elements from a single collection, returning only unique values. Internally, `Distinct()` maintains a `HashSet<T>` of elements it has already yielded — this makes it a **streaming** operator (it yields results one at a time without needing to read the entire source first), but with incrementally growing memory usage proportional to the number of unique elements.

By default, `Distinct()` uses the default equality comparer for the type (`EqualityComparer<T>.Default`). For primitive types like `int` and `string`, this works out of the box. For custom objects, you either need to override `Equals`/`GetHashCode` on the class, or pass in a custom `IEqualityComparer<T>`.

```csharp
var setA = new List<int> { 1, 2, 2, 3, 3, 3 };

var distinct = setA.Distinct();

foreach (var item in distinct)
{
    Console.WriteLine(item); // Output: 1, 2, 3
}
```

#### 7. Symmetric Difference

Returns elements that are in either set but **not in both** — the opposite of intersection. There is no built-in LINQ method for this, but it can be composed in two ways:

**Approach A** — `Union` minus `Intersect`:

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var symmetricDifference = setA.Union(setB).Except(setA.Intersect(setB));

foreach (var item in symmetricDifference)
{
    Console.WriteLine(item); // Output: 1, 2, 4, 5
}
```

**Approach B** — combine both `Except` directions (often more readable):

```csharp
var setA = new List<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

var symmetricDifference = setA.Except(setB).Union(setB.Except(setA));

foreach (var item in symmetricDifference)
{
    Console.WriteLine(item); // Output: 1, 2, 4, 5
}
```

Both approaches produce the same result. Approach B reads more naturally as "elements only in A, plus elements only in B." Note that neither approach is particularly efficient on large sequences since the source collections are enumerated multiple times — if performance matters, consider converting to `HashSet<T>` and using `SymmetricExceptWith()`.

```csharp
var hashA = new HashSet<int> { 1, 2, 3 };
var setB = new List<int> { 3, 4, 5 };

hashA.SymmetricExceptWith(setB); // Mutates hashA in place

foreach (var item in hashA)
{
    Console.WriteLine(item); // Output: 1, 2, 4, 5
}
```

### The `*By` Variants (.NET 6+) — Why They Exist

The set operations above (`Distinct`, `Union`, `Intersect`, `Except`) work perfectly on primitive types like `int` and `string`. But they **break on objects** because of how .NET compares reference types by default.

Consider this class:

```csharp
class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

When you call `Distinct()` on a list of `Product` objects, .NET uses **reference equality** — it asks "are these the exact same object in memory?", not "do these have the same property values?":

```csharp
var products = new List<Product>
{
    new Product { Name = "Apple", Price = 1.0m },
    new Product { Name = "Apple", Price = 1.0m },  // same data, different object
};

var distinct = products.Distinct().ToList();
Console.WriteLine(distinct.Count); // Output: 2 — Distinct() did nothing!
```

Two separate `new Product(...)` calls create two separate objects on the heap. Even though they contain identical data, `Distinct()` considers them different. The same problem hits `Union()`, `Intersect()`, and `Except()`.

Before .NET 6, the fixes were either inflexible or verbose:

| Approach | Downside |
|---|---|
| Override `Equals`/`GetHashCode` on the class | Permanently bakes **one** definition of equality into the type |
| Use a `record` instead of `class` | Same — one fixed definition, just less typing |
| Write a custom `IEqualityComparer<T>` | Flexible, but an **entire class** just to say "compare by Name" |

The `*By` methods solve this: you pass a lambda that tells LINQ **which property** to compare by. No class modification, no boilerplate, and you can swap the comparison property anytime.

#### 8. DistinctBy

Removes duplicates by comparing a **key** extracted from each element. Keeps the **first occurrence** of each key.

```csharp
var products = new List<Product>
{
    new("Apple", 1.0m),
    new("Banana", 0.5m),
    new("Apple", 1.5m),   // same name, different price
    new("Cherry", 1.0m),  // different name, same price
};

// Distinct by name — one product per name
var byName = products.DistinctBy(p => p.Name);
// Output: Apple (1.0), Banana (0.5), Cherry (1.0)
// Second Apple (1.5) dropped — "Apple" already seen

// Distinct by price — one product per price point
var byPrice = products.DistinctBy(p => p.Price);
// Output: Apple (1.0), Banana (0.5), Apple (1.5)
// Cherry (1.0) dropped — price 1.0 already seen
```

Same data, different lambda, completely different results. This flexibility is impossible with `Equals`/`GetHashCode` — you'd have to rewrite the class or create separate comparer classes for each.

#### 9. UnionBy

Combines two sequences and removes duplicates **by key**. Processes the first sequence entirely, then the second — so when duplicate keys appear across both, the element from the **first sequence wins**.

```csharp
var storeA = new List<Product>
{
    new("Laptop", 999.99m),
    new("Mouse", 29.99m),
};

var storeB = new List<Product>
{
    new("Mouse", 24.99m),   // same name, different price
    new("Webcam", 59.99m),
};

// Without UnionBy — broken (reference equality keeps all 4 items)
var broken = storeA.Union(storeB).ToList();
// Output: Laptop, Mouse (29.99), Mouse (24.99), Webcam — 4 items, duplicate Mouse!

// With UnionBy — works
var merged = storeA.UnionBy(storeB, p => p.Name);
// Output: Laptop (999.99), Mouse (29.99), Webcam (59.99) — 3 items
// Mouse comes from storeA (29.99) because first sequence wins
```

> [!note] **Order matters**
> `storeA.UnionBy(storeB, p => p.Name)` keeps storeA's Mouse ($29.99).
> `storeB.UnionBy(storeA, p => p.Name)` would keep storeB's Mouse ($24.99).
> Whichever sequence comes first takes priority on duplicate keys.

#### 10. IntersectBy

Keeps elements from the **first** sequence whose key appears in a set of keys you provide. Think of it as filtering your data against a checklist.

> [!warning] **Asymmetric signature**
> Unlike `UnionBy`, the second parameter is `IEnumerable<TKey>` (a list of **keys**), not `IEnumerable<TSource>` (a list of objects). The key selector only applies to the first sequence.

```csharp
var ourCatalog = new List<Product>
{
    new("Laptop", 999.99m),
    new("Mouse", 29.99m),
    new("Keyboard", 79.99m),
    new("Monitor", 349.99m),
};

// Scenario: a customer sends a wishlist of product names
string[] wishlistNames = ["Mouse", "Monitor", "Headphones"];

// Without IntersectBy — broken (reference equality finds zero matches)
// ourCatalog.Intersect(wishlistProducts) → empty list!

// With IntersectBy — works
var canFulfill = ourCatalog.IntersectBy(wishlistNames, p => p.Name);
// Output: Mouse (29.99), Monitor (349.99)
// Returns OUR Product objects with OUR prices
// "Headphones" is on the wishlist but not in our catalog — simply ignored
```

When you have two `Product` lists instead of a key list, extract the keys with `.Select()`:

```csharp
var wishlistProducts = new List<Product>
{
    new("Mouse", 0m),
    new("Monitor", 0m),
};

var canFulfill = ourCatalog.IntersectBy(
    wishlistProducts.Select(p => p.Name),  // extract keys: ["Mouse", "Monitor"]
    p => p.Name
);
// Same result: Mouse (29.99), Monitor (349.99) from ourCatalog
```

#### 11. ExceptBy

The inverse of `IntersectBy` — keeps elements from the **first** sequence whose key does **not** appear in the key set. Same asymmetric signature: second parameter is keys, not objects.

```csharp
var ourProducts = new List<Product>
{
    new("Laptop", 999.99m),
    new("Mouse", 29.99m),
    new("Keyboard", 79.99m),
    new("Monitor", 349.99m),
    new("Webcam", 59.99m),
};

// Scenario: find products we carry that a competitor doesn't
string[] competitorNames = ["Laptop", "Mouse", "Keyboard"];

// Without ExceptBy — broken (reference equality thinks everything is exclusive)
// ourProducts.Except(competitorProducts) → all 5 items! Useless.

// With ExceptBy — works
var exclusive = ourProducts.ExceptBy(competitorNames, p => p.Name);
// Output: Monitor (349.99), Webcam (59.99)
// These are the products only WE carry
```

When starting from two `Product` lists:

```csharp
var competitorProducts = new List<Product>
{
    new("Laptop", 899.99m),
    new("Mouse", 24.99m),
    new("Keyboard", 69.99m),
};

var exclusive = ourProducts.ExceptBy(
    competitorProducts.Select(p => p.Name),  // extract keys
    p => p.Name
);
// Same result: Monitor (349.99), Webcam (59.99)
```

> [!tip] **Symmetric vs. Asymmetric signatures**
> `DistinctBy` and `UnionBy` are **symmetric** — all parameters use the same element type, and the key selector applies to everything.
> `IntersectBy` and `ExceptBy` are **asymmetric** — the second parameter is a list of raw keys (`IEnumerable<TKey>`), not full objects. When you have two object collections, bridge with `.Select()` to extract the keys:
> ```csharp
> storeA.ExceptBy(storeB.Select(p => p.Name), p => p.Name);
> ```

### Practical Use Cases

1. **Filtering and Segmentation**: Easily segment data into different groups based on specific criteria.
2. **Data Comparison**: Compare datasets to find commonalities and differences, useful in data analysis and reporting.
3. **Database Queries**: Perform complex queries on database collections in a readable and maintainable way.

### Advantages of Using LINQ for Set Operations

1. **Readability**: LINQ provides a clear and concise syntax that closely resembles natural language, making it easier to understand and maintain.
2. **Expressiveness**: LINQ offers a rich set of operators that can be combined to perform complex queries and transformations.
3. **Integration**: LINQ integrates seamlessly with collections and databases in .NET, providing a unified approach to querying different data sources.
4. **Efficiency**: Many LINQ operations use deferred execution, meaning they only process data when needed, which can lead to performance optimizations.

### Summary

LINQ provides a powerful set of methods for performing operations that can be visualized as Venn diagrams, including union, intersection, difference, complement, and concatenation of sequences. These operations are efficient, readable, and integrate seamlessly with the .NET collections framework, making LINQ an excellent tool for data manipulation and analysis.

Using these LINQ operations, you can effectively manage and analyze sets of data, leveraging the expressive and concise syntax that LINQ offers. Whether working with in-memory collections or querying data from external sources, LINQ simplifies the process and enhances the capabilities of your data handling tasks.