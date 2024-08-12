## Null-conditional operators `?.` and `?[]`

A null-conditional operator applies a [member access](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#member-access-expression-) (`?.`) or [element access](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#indexer-operator-) (`?[]`) operation to its operand only if that operand evaluates to non-null; otherwise, it returns `null`. That is:

- If `a` evaluates to `null`, the result of `a?.x` or `a?[x]` is `null`.
    
- If `a` evaluates to non-null, the result of `a?.x` or `a?[x]` is the same as the result of `a.x` or `a[x]`, respectively.
    
    Note
    
    If `a.x` or `a[x]` throws an exception, `a?.x` or `a?[x]` would throw the same exception for non-null `a`. For example, if `a` is a non-null array instance and `x` is outside the bounds of `a`, `a?[x]` would throw an [IndexOutOfRangeException](https://learn.microsoft.com/en-us/dotnet/api/system.indexoutofrangeexception).
    

The null-conditional operators are short-circuiting. That is, if one operation in a chain of conditional member or element access operations returns `null`, the rest of the chain doesn't execute. In the following example, `B` isn't evaluated if `A` evaluates to `null` and `C` isn't evaluated if `A` or `B` evaluates to `null`:

C#

```csharp
A?.B?.Do(C);
A?.B?[C];
```

If `A` might be null but `B` and `C` wouldn't be null if A isn't null, you only need to apply the null-conditional operator to `A`:

C#

```csharp
A?.B.C();
```

In the preceding example, `B` isn't evaluated and `C()` isn't called if `A` is null. However, if the chained member access is interrupted, for example by parentheses as in `(A?.B).C()`, short-circuiting doesn't happen.

The following examples demonstrate the usage of the `?.` and `?[]` operators:

C#

```csharp
double SumNumbers(List<double[]> setsOfNumbers, int indexOfSetToSum)
{
    return setsOfNumbers?[indexOfSetToSum]?.Sum() ?? double.NaN;
}

var sum1 = SumNumbers(null, 0);
Console.WriteLine(sum1);  // output: NaN

List<double[]?> numberSets =
[
    [1.0, 2.0, 3.0],
    null
];

var sum2 = SumNumbers(numberSets, 0);
Console.WriteLine(sum2);  // output: 6

var sum3 = SumNumbers(numberSets, 1);
Console.WriteLine(sum3);  // output: NaN
```

C#

```csharp
namespace MemberAccessOperators2;

public static class NullConditionalShortCircuiting
{
    public static void Main()
    {
        Person? person = null;
        person?.Name.Write(); // no output: Write() is not called due to short-circuit.
        try
        {
            (person?.Name).Write();
        }
        catch (NullReferenceException)
        {
            Console.WriteLine("NullReferenceException");
        }; // output: NullReferenceException
    }
}

public class Person
{
    public required FullName Name { get; set; }
}

public class FullName
{
    public required string FirstName { get; set; }
    public required string LastName { get; set; }
    public void Write() => Console.WriteLine($"{FirstName} {LastName}");
}
```

The first of the preceding two examples also uses the [null-coalescing operator `??`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) to specify an alternative expression to evaluate in case the result of a null-conditional operation is `null`.

If `a.x` or `a[x]` is of a non-nullable value type `T`, `a?.x` or `a?[x]` is of the corresponding [nullable value type](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types) `T?`. If you need an expression of type `T`, apply the null-coalescing operator `??` to a null-conditional expression, as the following example shows:

C#

```csharp
int GetSumOfFirstTwoOrDefault(int[]? numbers)
{
    if ((numbers?.Length ?? 0) < 2)
    {
        return 0;
    }
    return numbers[0] + numbers[1];
}

Console.WriteLine(GetSumOfFirstTwoOrDefault(null));  // output: 0
Console.WriteLine(GetSumOfFirstTwoOrDefault([]));  // output: 0
Console.WriteLine(GetSumOfFirstTwoOrDefault([3, 4, 5]));  // output: 7
```

In the preceding example, if you don't use the `??` operator, `numbers?.Length < 2` evaluates to `false` when `numbers` is `null`.

Note

The `?.` operator evaluates its left-hand operand no more than once, guaranteeing that it cannot be changed to `null` after being verified as non-null.

The null-conditional member access operator `?.` is also known as the Elvis operator.