---
tags:
 - csharp
 - linq
---

LINQ was introduced in C# 3.0 alongside several language features that it depends on. Understanding these features individually is essential before studying LINQ, because LINQ query syntax and extension method syntax are built directly on top of them.

---

## 1. Implicitly Typed Local Variables (`var`)

The `var` keyword lets the compiler infer the type from the right-hand side of the assignment. LINQ needs this because query results often return [[Anonymous Types|anonymous types]] or complex generic types that would be impractical to write out explicitly.

```csharp
// Without var — you'd have to spell out the full generic type
IEnumerable<IGrouping<string, Employee>> grouped = employees.GroupBy(e => e.Department);

// With var — the compiler infers the same type
var grouped = employees.GroupBy(e => e.Department);
```

LINQ projections with `select new { ... }` **require** `var` because anonymous types have no writable name.

### Why LINQ Doesn't Just Return `IEnumerable<T>`

Most LINQ operators do return `IEnumerable<T>`, but some return a more specific type because that type **unlocks behavior** that `IEnumerable<T>` cannot express:

| Return type | Returned by | Why it exists |
|---|---|---|
| `IOrderedEnumerable<T>` | `OrderBy()`, `OrderByDescending()` | Enables `ThenBy()` / `ThenByDescending()` for secondary sorting — these methods are only defined on `IOrderedEnumerable<T>` |
| `IQueryable<T>` | EF / LINQ to SQL providers | Stores **expression trees** instead of delegates so the provider can translate C# into SQL. Falling back to `IEnumerable<T>` would pull the entire table into memory |
| `IGrouping<TKey, TElement>` | `GroupBy()` | Each group must expose its `.Key` — plain `IEnumerable<T>` has no concept of a key |
| Scalar types (`int`, `bool`, `T`) | `Count()`, `Any()`, `First()`, etc. | These compute a single value, not a sequence |

This variety of return types is another reason `var` is practical — you don't need to track whether the current chain returns `IEnumerable<T>`, `IOrderedEnumerable<T>`, or something else.

See also: [[Where you can not use implicit variable]]

---

## 2. Object and Collection Initialization Syntax

Object initializer syntax (`new Type { Prop = value }`) lets you set properties inline without writing a constructor call for each one. [[Collection Initialization Syntax|Collection initialization syntax]] (`new List<T> { ... }`) does the same for populating collections.

```csharp
// Object initializer — used inside LINQ Select projections
var projected = employees.Select(e => new EmployeeDto
{
    FullName = e.First + " " + e.Last,
    Department = e.Department
});

// Collection initializer — convenient for building test data to query
var employees = new List<Employee>
{
    new Employee { First = "Long", Last = "Pham", Department = "IT" },
    new Employee { First = "Jane", Last = "Smith", Department = "HR" }
};
```

Without these, every LINQ projection or inline data setup would require verbose constructor calls or multi-line property assignments.

See also: [[Constructor#Default Constructor and Object Initializers]]

---

## 3. Lambda Expressions

[[Lambda Expression Overview|Lambda expressions]] (`=>`) are the primary way you pass logic into LINQ operators. Every `Where`, `Select`, `OrderBy`, etc. takes a delegate — and lambdas are the concise syntax for defining that delegate inline.

```csharp
// Lambda as a predicate (Func<Employee, bool>)
var seniors = employees.Where(e => e.YearsOfExperience > 5);

// Lambda as a projection (Func<Employee, anonymous type>)
var names = employees.Select(e => new { e.First, e.Last });

// Lambda as a key selector (Func<Employee, string>)
var sorted = employees.OrderBy(e => e.Last);
```

Lambdas replaced the older [[Anonymous Method Overview|anonymous method]] syntax (`delegate { ... }`) and are effectively syntactic sugar over it.

---

## 4. Extension Methods

[[Extension Methods]] are what make LINQ's fluent chaining syntax possible. All the standard query operators (`Where`, `Select`, `OrderBy`, `GroupBy`, etc.) are **extension methods** on `IEnumerable<T>` defined in the `System.Linq.Enumerable` class.

```csharp
// This chain is only possible because each method is an extension method on IEnumerable<T>
var result = employees
    .Where(e => e.Department == "IT")
    .OrderBy(e => e.Last)
    .Select(e => new { e.First, e.Last });

// What the compiler actually emits:
var result = Enumerable.Select(
    Enumerable.OrderBy(
        Enumerable.Where(employees, e => e.Department == "IT"),
        e => e.Last),
    e => new { e.First, e.Last });
```

Without extension methods, you'd have to nest static method calls — the fluent syntax disappears entirely.

---

## 5. Anonymous Types

[[Anonymous Types]] (`new { ... }`) let you create lightweight, read-only objects without declaring a class. LINQ uses them heavily in projections when you need a shape that doesn't match any existing type.

```csharp
// Anonymous type in a Select projection
var result = employees.Select(e => new { e.First, e.Department });

// The compiler generates a real class with read-only properties
// and value-based Equals/GetHashCode — which is why Distinct() works:
var unique = employees
    .Select(e => new { e.Department, e.Location })
    .Distinct();
```

Anonymous types are confined to local scope — you can't return them from a method. For cross-boundary use, prefer tuples or records.

---

## How They All Connect

These five features work together to make LINQ possible:

```
Lambda Expressions ──→ provide inline logic to query operators
Extension Methods  ──→ enable fluent .Where().Select().OrderBy() chaining
Anonymous Types    ──→ allow ad-hoc projection shapes in Select
var                ──→ captures results when the type is inferred or anonymous
Object/Collection  ──→ build inline data and projected objects cleanly
  Initialization
```

A typical LINQ query uses all five at once:

```csharp
var result = employees                                    // var (implicitly typed)
    .Where(e => e.Age > 30)                               // extension method + lambda
    .Select(e => new { e.First, e.Last, e.Department })   // extension method + lambda + anonymous type
    .ToList();                                             // extension method
```

Without any one of these features, LINQ's syntax would be significantly more verbose or impossible in its current form. They were all introduced together in C# 3.0 specifically to enable LINQ.
