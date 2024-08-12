Aggregation operations in LINQ allow you to perform calculations over a sequence of data, producing a single value that represents some characteristic of the data set. These operations include methods like `Count`, `Sum`, `Average`, `Min`, `Max`, and `Aggregate`. Each of these methods serves a different purpose and can be used to derive meaningful insights from your data.

### Common LINQ Aggregation Methods

1. **Count**: Counts the number of elements in the sequence.
2. **Sum**: Computes the sum of the elements in the sequence.
3. **Average**: Computes the average of the elements in the sequence.
4. **Min**: Finds the minimum value in the sequence.
5. **Max**: Finds the maximum value in the sequence.
6. **Aggregate**: Performs a custom aggregation operation on the sequence.

### Examples of Aggregation Operations

#### 1. Count

Counts the number of elements in a collection.

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };
var count = numbers.Count();
Console.WriteLine(count); // Output: 5
```

You can also use `Count` with a predicate to count elements that match a certain condition.

```csharp
var countEven = numbers.Count(n => n % 2 == 0);
Console.WriteLine(countEven); // Output: 2
```

#### 2. Sum

Computes the sum of the elements in a collection.

```csharp
var sum = numbers.Sum();
Console.WriteLine(sum); // Output: 15
```

You can also sum specific properties in a collection of objects.

```csharp
var people = new List<Person>
{
    new Person { Id = 1, Age = 30 },
    new Person { Id = 2, Age = 25 }
};

var totalAge = people.Sum(p => p.Age);
Console.WriteLine(totalAge); // Output: 55
```

#### 3. Average

Computes the average of the elements in a collection.

```csharp
var average = numbers.Average();
Console.WriteLine(average); // Output: 3
```

Similarly, you can compute the average of a specific property.

```csharp
var averageAge = people.Average(p => p.Age);
Console.WriteLine(averageAge); // Output: 27.5
```

#### 4. Min

Finds the minimum value in a collection.

```csharp
var min = numbers.Min();
Console.WriteLine(min); // Output: 1
```

Or find the minimum of a specific property.

```csharp
var minAge = people.Min(p => p.Age);
Console.WriteLine(minAge); // Output: 25
```

#### 5. Max

Finds the maximum value in a collection.

```csharp
var max = numbers.Max();
Console.WriteLine(max); // Output: 5
```

Or find the maximum of a specific property.

```csharp
var maxAge = people.Max(p => p.Age);
Console.WriteLine(maxAge); // Output: 30
```

#### 6. Aggregate

Performs a custom aggregation operation. This is a more general and powerful aggregation method that allows you to define how the aggregation is performed.

```csharp
var product = numbers.Aggregate((acc, n) => acc * n);
Console.WriteLine(product); // Output: 120
```

You can also provide a seed value.

```csharp
var sumWithSeed = numbers.Aggregate(10, (acc, n) => acc + n);
Console.WriteLine(sumWithSeed); // Output: 25
```

### Summary

Aggregation operations in LINQ provide a powerful way to summarize and analyze data. The common methods such as `Count`, `Sum`, `Average`, `Min`, `Max`, and `Aggregate` cover a wide range of scenarios, from simple counting to complex custom calculations. These methods are essential tools in any developer's toolkit for working with collections in a concise and expressive manner.