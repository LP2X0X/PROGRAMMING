---
tags:
  - solid
  - lsp
  - liskov-substitution
---

## 🔹 Definition

The **Liskov Substitution Principle (LSP)** is the third principle in [[SOLID Overview|SOLID]]. It was first articulated by **Barbara Liskov** in her 1987 keynote address at the OOPSLA conference, then formalized jointly with **Jeannette Wing** in a 1994 paper titled *"A Behavioral Notion of Subtyping."*

The formal statement:

> **If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program (correctness, task performed, etc.).**

In plain language: ==if you have code that works with a base class (or interface), you should be able to hand it any derived class (or implementer) and it must still work correctly -- no surprises, no exceptions the base didn't throw, no violated assumptions.== The subtype must honor the *behavioral contract* of the supertype, not just its structural signature.

This is subtler than it sounds. Compiling successfully does not mean LSP compliance. The compiler checks that method signatures match -- that overrides have the right return type, the right parameters. It does **not** check that the behavior of those methods respects the expectations established by the base class. LSP is about *behavioral compatibility*, which is a semantic property that the type system cannot enforce.

> [!info] Definition
> **Behavioral subtyping** means that a subtype preserves the *observable behavior* and *contractual guarantees* of its supertype. A `Square` is a mathematical subtype of `Rectangle` (every square is a rectangle), but it is **not** a behavioral subtype if it violates the expectation that width and height can be set independently.

The key insight: **"IS-A" in the mathematical or real-world sense does not automatically mean "IS-A" in the behavioral, programmatic sense.** Inheritance models behavioral substitutability, not taxonomic classification.

---

## 🔹 The Four Contract Rules (Behavioral Subtyping)

LSP is enforced through four rules that govern the relationship between a base type's contract and any derived type's behavior. These rules are sometimes called the **Design by Contract rules** (from Bertrand Meyer's work, which heavily influenced Liskov's formalization). A subtype must obey ALL of these for substitutability to hold.

| Rule | What It Means | Violation Example |
|---|---|---|
| **Preconditions cannot be strengthened** | A derived class must accept at least everything the base class accepts. It cannot add new requirements the caller must satisfy before calling the method. | Base `Withdraw(decimal amount)` accepts any positive amount. Derived `FixedDepositAccount.Withdraw()` requires the maturity date to have passed. Callers that withdraw before maturity now fail. |
| **Postconditions cannot be weakened** | A derived class must guarantee at least everything the base class guarantees after a method completes. It cannot promise less. | Base `GetAll()` guarantees a non-null collection. Derived class returns `null` for empty results instead of an empty collection. Callers that iterate without null-checking now crash. |
| **Invariants must be preserved** | Properties that are always true about the base class must remain true in the derived class throughout the object's lifetime. | Base `Rectangle` invariant: `Area == Width * Height` with width and height independently settable. `Square` breaks this by coupling width and height -- setting width changes height, making `Area` unpredictable for code that sets them sequentially. |
| **History constraint** | A derived class must not introduce state changes that the base class could never exhibit. The observable history of state transitions must be a valid history for the base type. | Base `ImmutablePoint` never changes coordinates after construction. Derived `MutablePoint` adds `SetX()`/`SetY()` methods. Code holding an `ImmutablePoint` reference assumes coordinates are stable -- the derived class violates that assumption. |

### Preconditions Cannot Be Strengthened

A **precondition** is a condition that must be true *before* a method is called. It defines what inputs the method considers valid. When a base class method accepts any positive `decimal`, a derived class cannot narrow that to "only multiples of 100" or "only amounts above $50." The caller has no way to know about these extra requirements if it is working through a base-type reference.

```csharp
public class BankAccount
{
    public decimal Balance { get; protected set; }

    public BankAccount(decimal initialBalance)
    {
        Balance = initialBalance;
    }

    // Precondition: amount > 0 and amount <= Balance
    public virtual void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds.");

        Balance -= amount;
    }
}

public class MinimumBalanceAccount : BankAccount
{
    private readonly decimal _minimumBalance;

    public MinimumBalanceAccount(decimal initialBalance, decimal minimumBalance)
        : base(initialBalance)
    {
        _minimumBalance = minimumBalance;
    }

    // VIOLATION: strengthened precondition -- now requires Balance - amount >= _minimumBalance
    // Callers using a BankAccount reference have no idea about this extra constraint
    public override void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        if (Balance - amount < _minimumBalance)
            throw new InvalidOperationException(
                $"Cannot withdraw. Balance would drop below minimum of {_minimumBalance:C}.");

        Balance -= amount;
    }
}

// Caller code that works with BankAccount:
public void WithdrawFromAccount(BankAccount account)
{
    if (account.Balance >= 100m)
    {
        // Caller checked the base class precondition: amount > 0 && amount <= Balance
        // But MinimumBalanceAccount ALSO requires Balance - amount >= minimumBalance
        // This will unexpectedly throw for MinimumBalanceAccount
        account.Withdraw(100m);
    }
}
```

The caller satisfied every precondition documented on `BankAccount`, but `MinimumBalanceAccount` added a secret extra precondition. The caller cannot know about it because it is working through a `BankAccount` reference.

Note: a derived class **can** *loosen* preconditions (accept more than the base). This is safe because callers who satisfy the base precondition automatically satisfy a weaker one.

### Postconditions Cannot Be Weakened

A **postcondition** is a condition that the method guarantees to be true *after* it returns. If the base class's `Search()` guarantees it returns a non-null `IEnumerable<T>` (possibly empty), the derived class cannot return `null`:

```csharp
public class SearchService
{
    protected readonly List<Product> _products = new();

    // Postcondition: always returns a non-null collection (may be empty)
    public virtual IEnumerable<Product> Search(string query)
    {
        if (string.IsNullOrWhiteSpace(query))
            return Enumerable.Empty<Product>();

        return _products.Where(p =>
            p.Name.Contains(query, StringComparison.OrdinalIgnoreCase));
    }
}

public class CachedSearchService : SearchService
{
    private readonly Dictionary<string, IEnumerable<Product>?> _cache = new();

    // VIOLATION: weakened postcondition -- might return null
    public override IEnumerable<Product> Search(string query)
    {
        if (_cache.TryGetValue(query, out var cached))
            return cached!; // could be null if the cache stored null

        var results = base.Search(query);
        _cache[query] = results;
        return results;
    }
}

// Caller code relies on the postcondition:
public void DisplayResults(SearchService service, string query)
{
    var results = service.Search(query);
    // Caller trusts the postcondition: results is never null
    Console.WriteLine($"Found {results.Count()} items."); // NullReferenceException if null!
}
```

Note: a derived class **can** *strengthen* postconditions (guarantee more than the base). For example, if the base guarantees a non-null return, the derived class can additionally guarantee the collection is sorted. Extra guarantees never break callers who only rely on the base's guarantees.

### Invariants Must Be Preserved

An **invariant** is a condition that is always true for an object throughout its lifetime -- at the beginning and end of every public method call. For a `Rectangle`, the invariant is that width and height are independently settable and `Area == Width * Height` always holds under independent assignment. This is the core of the classic Square/Rectangle violation (detailed in the next section).

```csharp
public class PositiveBalance
{
    public decimal Balance { get; protected set; }

    // INVARIANT: Balance >= 0 at all times
    public PositiveBalance(decimal initialBalance)
    {
        if (initialBalance < 0)
            throw new ArgumentException("Balance cannot be negative.");
        Balance = initialBalance;
    }

    public virtual void Debit(decimal amount)
    {
        if (amount > Balance)
            throw new InvalidOperationException("Would violate positive balance invariant.");
        Balance -= amount;
    }
}

public class OverdraftAccount : PositiveBalance
{
    // VIOLATION: allows Balance to go negative, breaking the invariant
    public override void Debit(decimal amount)
    {
        Balance -= amount; // no guard -- Balance can now be negative!
    }
}
```

### History Constraint (The History Rule)

The **history constraint** is the subtlest rule. It says that the set of possible state transitions of a derived object, when observed through a base-type reference, must be a subset of the state transitions possible for the base type. If the base type is immutable, the derived type cannot be mutable:

```csharp
public class ImmutablePoint
{
    public int X { get; }
    public int Y { get; }

    public ImmutablePoint(int x, int y)
    {
        X = x;
        Y = y;
    }
}

public class MutablePoint : ImmutablePoint
{
    // VIOLATION: introduces state changes the base type cannot exhibit
    public new int X { get; set; }
    public new int Y { get; set; }

    public MutablePoint(int x, int y) : base(x, y)
    {
        X = x;
        Y = y;
    }
}

// Code that depends on immutability:
public void CacheExpensiveComputation(ImmutablePoint point)
{
    // Trusts that point.X and point.Y will never change (immutability invariant)
    _cache[point] = ComputeExpensiveValue(point.X, point.Y);
    // If point is actually a MutablePoint, someone could change X/Y later,
    // making the cache entry stale and the computed value wrong
}
```

> [!warning] Common Misconception
> Many developers think LSP is just about "a derived class should work where the base class works." That is only the surface. LSP is specifically about **contracts** -- preconditions, postconditions, invariants, and history. Two classes can have identical method signatures and still violate LSP if they do not honor the same behavioral contracts. Compilation is necessary but not sufficient.

---

## 🔹 Classic Violation: Square Extends Rectangle

This is the most famous LSP violation in all of software engineering. It illustrates the fundamental tension between mathematical IS-A relationships and behavioral IS-A relationships in programming.

### The Naive Implementation

Mathematically, a square is a rectangle (a rectangle where width equals height). So it seems natural to model `Square` as a subclass of `Rectangle`:

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int Area => Width * Height;

    public override string ToString()
    {
        return $"Rectangle(Width={Width}, Height={Height}, Area={Area})";
    }
}

public class Square : Rectangle
{
    // A square must keep width and height synchronized.
    // Override the setters to enforce this constraint.
    public override int Width
    {
        get => base.Width;
        set
        {
            base.Width = value;
            base.Height = value; // sync height to match width
        }
    }

    public override int Height
    {
        get => base.Height;
        set
        {
            base.Width = value; // sync width to match height
            base.Height = value;
        }
    }

    public override string ToString()
    {
        return $"Square(Side={Width}, Area={Area})";
    }
}
```

This compiles perfectly. The `Square` IS-A `Rectangle` as far as the compiler is concerned. Every method on `Rectangle` exists on `Square` with compatible signatures.

### The Test That Breaks

Now write code that uses `Rectangle` polymorphically:

```csharp
public static class RectangleResizer
{
    /// <summary>
    /// Sets a rectangle to 5x10 and verifies the area.
    /// Relies on the Rectangle contract: width and height are independent.
    /// </summary>
    public static void Resize(Rectangle r)
    {
        r.Width = 5;
        r.Height = 10;

        // Rectangle contract: Area = Width * Height = 5 * 10 = 50
        Debug.Assert(r.Area == 50,
            $"Expected area 50 but got {r.Area}. " +
            $"Width={r.Width}, Height={r.Height}");
    }
}

// Test with Rectangle -- PASSES
var rect = new Rectangle();
RectangleResizer.Resize(rect);
// After r.Width = 5:   Width=5, Height=0 (default)
// After r.Height = 10: Width=5, Height=10
// Area = 5 * 10 = 50 ✓

// Test with Square -- FAILS
var square = new Square();
RectangleResizer.Resize(square);
// After r.Width = 5:   Width=5, Height=5   (Height synced to 5)
// After r.Height = 10: Width=10, Height=10 (Width synced to 10)
// Area = 10 * 10 = 100 ≠ 50 ✗ -- ASSERTION FAILS
```

The `Resize` method follows `Rectangle`'s contract perfectly: it sets width and height independently and expects the area to equal their product. But `Square` secretly couples width and height -- setting one overwrites the other -- violating the independence invariant.

### Why This Is an LSP Violation

The confusion arises because **mathematical subtyping** (set theory: squares are a subset of rectangles) does not equal **behavioral subtyping** (programmatic: Square instances can substitute for Rectangle instances without breaking anything).

**In mathematics:**
- A square is a rectangle (with the additional constraint that width == height).
- The set of all squares is a proper subset of the set of all rectangles.
- This is a static, declarative relationship about properties.

**In programming:**
- A `Rectangle` establishes a behavioral contract: `Width` and `Height` are independently mutable, and `Area` is their product.
- A `Square` cannot honor this contract because its essential constraint (width == height) conflicts with the independence guarantee.
- The behavioral relationship is dynamic -- it concerns how the object responds to messages (method calls) over time.

The `Rectangle` has an **implicit contract** that callers rely on:
- Setting `Width` changes only `Width` (postcondition: `Height` is unchanged)
- Setting `Height` changes only `Height` (postcondition: `Width` is unchanged)

`Square` violates both postconditions. After `r.Width = 5`, the caller expects `Height` to still be whatever it was before. But `Square` silently changed it to 5.

==Inheritance in OOP models behavioral substitutability, not taxonomic classification. "IS-A" in code means "CAN SUBSTITUTE FOR," not "IS A SUBSET OF."==

> [!warning] Common Misconception
> "A square is a rectangle, so `Square` should extend `Rectangle`." This is the most persistent misconception in OOP design. The mathematical relationship is irrelevant. What matters is whether a `Square` can do everything a `Rectangle` can do, in the same way, without surprises. It cannot -- because independently setting width and height is fundamental to what a `Rectangle` promises.

### Fix 1: Remove the Inheritance Entirely

If `Square` and `Rectangle` have different behavioral contracts, they should not be in a parent-child relationship:

```csharp
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area => Width * Height;
}

public class Square
{
    public int Side { get; set; }
    public int Area => Side * Side;
}
```

Simple, honest, and impossible to misuse. Each class has its own contract. There is no polymorphic relationship to violate.

### Fix 2: Shared Interface with Read-Only Properties

If you genuinely need polymorphism (e.g., computing area for a collection of shapes), define a shared interface that only exposes what all shapes have in common:

```csharp
public interface IShape
{
    int Area { get; }
    double Perimeter { get; }
}

public interface IRectangular : IShape
{
    int Width { get; }
    int Height { get; }
}

public class Rectangle : IRectangular
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area => Width * Height;
    public double Perimeter => 2 * (Width + Height);
}

public class Square : IShape
{
    public int Side { get; set; }
    public int Area => Side * Side;
    public double Perimeter => 4 * Side;
}

// Consumer that needs area -- works for both, no LSP concerns
public static int TotalArea(IEnumerable<IShape> shapes)
{
    return shapes.Sum(s => s.Area);
}
```

Note that `Square` does **not** implement `IRectangular` because it cannot honestly expose independent `Width` and `Height`. It only implements `IShape`, which requires nothing that `Square` cannot deliver.

### Fix 3: Composition

If a `Square` needs to interoperate with rectangle-focused code, compose rather than inherit:

```csharp
public class Square
{
    public int Side { get; set; }
    public int Area => Side * Side;

    /// <summary>
    /// Creates a Rectangle snapshot with the current side length.
    /// This is a one-time conversion, not a live link.
    /// </summary>
    public Rectangle ToRectangle()
    {
        return new Rectangle { Width = Side, Height = Side };
    }
}
```

### Fix 4: Immutable Rectangle

Another approach is to make `Rectangle` immutable, eliminating the mutation-based contract:

```csharp
public class ImmutableRectangle
{
    public int Width { get; }
    public int Height { get; }
    public int Area => Width * Height;

    public ImmutableRectangle(int width, int height)
    {
        Width = width;
        Height = height;
    }

    // Returns a NEW rectangle -- the original is unchanged
    public ImmutableRectangle WithWidth(int newWidth) => new(newWidth, Height);
    public ImmutableRectangle WithHeight(int newHeight) => new(Width, newHeight);
}
```

With immutability, there is no mutable state for `Square` to corrupt. However, `Square` still cannot inherit from this because `WithWidth(5)` on a square of side 10 would return a non-square rectangle (5x10) -- the inheritance relationship still does not make behavioral sense.

---

## 🔹 More Violations

### ReadOnlyCollection Implementing IList\<T\>

`System.Collections.ObjectModel.ReadOnlyCollection<T>` implements `IList<T>`, which includes `Add()`, `Remove()`, `Insert()`, `RemoveAt()`, and the settable indexer `this[int index] { set; }`. Every one of these mutation methods throws `NotSupportedException` at runtime:

```csharp
IList<string> list = new ReadOnlyCollection<string>(new[] { "a", "b", "c" });

// All of these compile but throw NotSupportedException at runtime:
list.Add("d");            // throws
list.Remove("a");         // throws
list.Insert(0, "z");      // throws
list.RemoveAt(1);         // throws
list[0] = "modified";     // throws
```

Code that accepts `IList<T>` and calls mutation methods will break when handed a `ReadOnlyCollection<T>`. The contract of `IList<T>` implies that those methods work. `ReadOnlyCollection<T>` violates this contract by making them time bombs.

This is a well-known design compromise in the .NET BCL -- `IList<T>` includes `IsReadOnly` so callers *can* check, but most callers do not. The .NET team later addressed this by introducing `IReadOnlyList<T>` in .NET 4.5, which is the proper [[4 - Interface Segregation Principle|ISP]]-compliant solution:

```csharp
// ISP-compliant: expose only what ReadOnlyCollection can honestly deliver
IReadOnlyList<string> readOnlyList = new ReadOnlyCollection<string>(new[] { "a", "b", "c" });

// readOnlyList.Add("d");  -- COMPILE ERROR. Method does not exist.
// The type system prevents the violation entirely.
string first = readOnlyList[0]; // works fine -- read-only indexer
int count = readOnlyList.Count; // works fine
```

> [!tip] Practical Tip
> Whenever you are accepting a collection parameter and you only need to read from it, use `IReadOnlyList<T>`, `IReadOnlyCollection<T>`, or `IEnumerable<T>` -- not `IList<T>` or `ICollection<T>`. This communicates your intent to the caller, prevents accidental mutation, and avoids LSP traps.

### Bird.Fly() with Penguin

The classic animal hierarchy trap. A `Bird` base class with a `Fly()` method seems reasonable -- until you need a `Penguin`:

```csharp
public abstract class Bird
{
    public string Name { get; }

    protected Bird(string name) => Name = name;

    public abstract void Eat();
    public abstract void Fly(); // Not all birds can fly!
}

public class Eagle : Bird
{
    public Eagle() : base("Eagle") { }

    public override void Eat() => Console.WriteLine("Eagle tears meat with its beak.");
    public override void Fly() => Console.WriteLine("Eagle soars at high altitude.");
}

public class Penguin : Bird
{
    public Penguin() : base("Penguin") { }

    public override void Eat() => Console.WriteLine("Penguin swallows fish whole.");

    // LSP VIOLATION: Throws an exception the base class does not document.
    public override void Fly()
    {
        throw new NotSupportedException("Penguins can't fly!");
    }
}

// Consumer code:
public void MigrateFlock(List<Bird> flock, string destination)
{
    Console.WriteLine($"Migrating to {destination}...");
    foreach (var bird in flock)
    {
        bird.Fly(); // Crashes when it hits a Penguin!
    }
}
```

**The fix:** Separate the flying capability from the bird identity using [[4 - Interface Segregation Principle|ISP]]:

```csharp
public interface IBird
{
    string Name { get; }
    void Eat();
    void MakeSound();
}

public interface IFlyable
{
    void Fly();
    double MaxAltitudeFeet { get; }
}

public interface ISwimmable
{
    void Swim();
    double MaxDepthFeet { get; }
}

public class Eagle : IBird, IFlyable
{
    public string Name => "Eagle";
    public double MaxAltitudeFeet => 10000;

    public void Eat() => Console.WriteLine("Eagle tears meat with its beak.");
    public void MakeSound() => Console.WriteLine("Eagle screeches.");
    public void Fly() => Console.WriteLine("Eagle soars at high altitude.");
}

public class Penguin : IBird, ISwimmable
{
    public string Name => "Penguin";
    public double MaxDepthFeet => 1850;

    public void Eat() => Console.WriteLine("Penguin swallows fish whole.");
    public void MakeSound() => Console.WriteLine("Penguin honks.");
    public void Swim() => Console.WriteLine("Penguin dives deep underwater.");
}

// Only asks for IFlyable -- Penguin is never passed here
public void MigrateFlock(IEnumerable<IFlyable> flock, string destination)
{
    Console.WriteLine($"Migrating to {destination}...");
    foreach (var flyer in flock)
    {
        flyer.Fly(); // Safe: everything in this list CAN fly
    }
}

// Works with all birds -- only uses IBird contract
public void FeedAllBirds(IEnumerable<IBird> birds)
{
    foreach (var bird in birds)
    {
        Console.Write($"{bird.Name}: ");
        bird.Eat(); // Safe: all IBird implementations have Eat()
    }
}
```

Now `Penguin` never has to fake `Fly()`. The type system prevents it from being passed to code that requires flying. No runtime exceptions, no empty methods.

### Account.Withdraw() with FixedDeposit

A banking system with a base `Account` class where `FixedDepositAccount` restricts withdrawals:

```csharp
public class Account
{
    public string AccountNumber { get; }
    public decimal Balance { get; protected set; }

    public Account(string accountNumber, decimal initialBalance)
    {
        AccountNumber = accountNumber;
        Balance = initialBalance;
    }

    // Contract: any positive amount up to Balance can be withdrawn
    public virtual void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds.");

        Balance -= amount;
        Console.WriteLine($"Withdrew {amount:C} from {AccountNumber}. Balance: {Balance:C}");
    }

    public virtual void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));

        Balance += amount;
    }
}

public class FixedDepositAccount : Account
{
    public DateTime MaturityDate { get; }

    public FixedDepositAccount(string accountNumber, decimal initialBalance,
        DateTime maturityDate)
        : base(accountNumber, initialBalance)
    {
        MaturityDate = maturityDate;
    }

    // VIOLATION: strengthened precondition!
    // Base class says: just need positive amount <= Balance
    // This subtype adds: AND maturity date must have passed
    // AND amount must equal full balance
    public override void Withdraw(decimal amount)
    {
        if (DateTime.UtcNow < MaturityDate)
            throw new InvalidOperationException(
                $"Cannot withdraw before maturity date ({MaturityDate:d}).");

        if (amount != Balance)
            throw new InvalidOperationException(
                "Fixed deposit accounts require full balance withdrawal.");

        base.Withdraw(amount);
    }
}

// Consumer code that works with any Account:
public void ProcessEmergencyWithdrawal(Account account, decimal amount)
{
    // Follows the base Account contract: positive amount, not exceeding balance
    if (amount > 0 && amount <= account.Balance)
    {
        account.Withdraw(amount); // EXPLODES for FixedDepositAccount
    }
}
```

The caller satisfied every precondition documented on `Account`, but `FixedDepositAccount` added secret extra preconditions (maturity date passed, must withdraw full balance). The calling code cannot know about these constraints when working through an `Account` reference.

**The fix:** Do not inherit `FixedDepositAccount` from `Account`. Use separate interfaces for different withdrawal behaviors:

```csharp
public interface IAccount
{
    string AccountNumber { get; }
    decimal Balance { get; }
    void Deposit(decimal amount);
}

public interface IWithdrawable
{
    bool CanWithdraw(decimal amount);
    void Withdraw(decimal amount);
}

public interface IMaturable
{
    DateTime MaturityDate { get; }
    bool IsMatured { get; }
    void Redeem();
}

public class SavingsAccount : IAccount, IWithdrawable
{
    public string AccountNumber { get; }
    public decimal Balance { get; private set; }

    public SavingsAccount(string accountNumber, decimal initialBalance)
    {
        AccountNumber = accountNumber;
        Balance = initialBalance;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        Balance += amount;
    }

    public bool CanWithdraw(decimal amount) => amount > 0 && amount <= Balance;

    public void Withdraw(decimal amount)
    {
        if (!CanWithdraw(amount))
            throw new InvalidOperationException("Invalid amount or insufficient funds.");
        Balance -= amount;
    }
}

public class FixedDepositAccount : IAccount, IMaturable
{
    public string AccountNumber { get; }
    public decimal Balance { get; private set; }
    public DateTime MaturityDate { get; }
    public bool IsMatured => DateTime.UtcNow >= MaturityDate;

    public FixedDepositAccount(string accountNumber, decimal initialBalance,
        DateTime maturityDate)
    {
        AccountNumber = accountNumber;
        Balance = initialBalance;
        MaturityDate = maturityDate;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        Balance += amount;
    }

    public void Redeem()
    {
        if (!IsMatured)
            throw new InvalidOperationException(
                $"Cannot redeem before maturity date ({MaturityDate:d}).");

        decimal amount = Balance;
        Balance = 0;
        Console.WriteLine($"Redeemed {amount:C} from {AccountNumber}.");
    }
}

// Consumer code -- only IWithdrawable accounts can be withdrawn from
public void ProcessEmergencyWithdrawal(IWithdrawable account, decimal amount)
{
    if (account.CanWithdraw(amount))
        account.Withdraw(amount); // Safe: every IWithdrawable supports this
}

// FixedDepositAccount is not IWithdrawable -- compiler prevents misuse
```

`FixedDepositAccount` never implements `IWithdrawable` because it cannot honor that contract. The compiler enforces the business rule.

---

## 🔹 How to Detect LSP Violations

### 1. Derived Class Throws Exceptions the Base Does Not

The most obvious red flag. If a base class method never throws `NotSupportedException` or `NotImplementedException`, but a derived class override does, the derived class is breaking the postcondition that the method completes normally for valid inputs:

```csharp
// Red flag: any method body that looks like this
public override void SomeMethod()
{
    throw new NotSupportedException("This operation is not supported.");
}

public override void AnotherMethod()
{
    throw new NotImplementedException();
}
```

**Audit rule:** grep your codebase for `NotSupportedException` and `NotImplementedException` inside `override` methods. Every hit is a potential LSP violation.

### 2. Empty Overrides (No-Ops)

Sometimes worse than throwing because they silently do nothing. The caller expects an action; the override provides none. The bug is invisible until data is lost or behavior is missing:

```csharp
public override void Save()
{
    // do nothing -- this type doesn't support saving
}

public override void Validate()
{
    // intentionally left blank
}
```

The caller calls `Save()` believing data was persisted. It was not. The bug is invisible until data is lost.

> [!warning] Common Misconception
> "A no-op is safer than throwing." Not true. Throwing at least makes the problem visible. A no-op makes the problem invisible -- the caller has no way to know the operation silently did nothing. Silent failures are harder to diagnose and more dangerous in production than loud failures. The one exception is the **Null Object pattern** (e.g., `NullLogger`) which is an intentional no-op and must be documented as such.

### 3. Type-Checking in Callers

When calling code checks the concrete type of an object before deciding what to do, it means the subtypes do not behave uniformly -- LSP is violated:

```csharp
// Huge red flag: caller must know about specific subtypes
public void ProcessShape(Shape shape)
{
    if (shape is Circle circle)
    {
        // special handling for circles
    }
    else if (shape is Square square)
    {
        // special handling for squares -- can't treat it like a rectangle
    }
    else if (shape is Rectangle rect)
    {
        // general rectangle handling
    }
}
```

If LSP were satisfied, all shapes could be processed uniformly through the `Shape` interface. The `if/else` chain based on `is` or `as` casts is evidence that the subtypes do not honor a common contract.

Note: `is` checks are fine when you are doing *optional* capability detection (e.g., checking if a stream supports seeking with `if (stream.CanSeek)`). They become an LSP smell when they are *required* -- when the code **breaks** if you do not check, meaning the subtype cannot substitute properly.

### 4. Tests Pass for Base, Fail for Derived

The most reliable detection method. Write a test suite against the base class contract. Then run the exact same tests with every derived class instance. If any test fails for a derived class, that class violates LSP:

```csharp
public abstract class AccountContractTests<T> where T : Account
{
    protected abstract T CreateAccount(decimal initialBalance);

    [Fact]
    public void Withdraw_ValidAmount_ReducesBalance()
    {
        var account = CreateAccount(1000m);
        account.Withdraw(200m);
        Assert.Equal(800m, account.Balance);
    }

    [Fact]
    public void Withdraw_EntireBalance_LeavesZero()
    {
        var account = CreateAccount(500m);
        account.Withdraw(500m);
        Assert.Equal(0m, account.Balance);
    }

    [Fact]
    public void Withdraw_ExceedingBalance_Throws()
    {
        var account = CreateAccount(100m);
        Assert.Throws<InvalidOperationException>(() => account.Withdraw(200m));
    }
}

// Passes for SavingsAccount:
public class SavingsAccountTests : AccountContractTests<SavingsAccount>
{
    protected override SavingsAccount CreateAccount(decimal initialBalance)
        => new SavingsAccount("SAV001", initialBalance);
}

// FAILS for FixedDepositAccount -- Withdraw_ValidAmount_ReducesBalance throws
// because FixedDepositAccount adds preconditions the base doesn't have
public class FixedDepositAccountTests : AccountContractTests<FixedDepositAccount>
{
    protected override FixedDepositAccount CreateAccount(decimal initialBalance)
        => new FixedDepositAccount("FD001", initialBalance, DateTime.UtcNow.AddYears(1));
}
```

This is the **definitive** test for LSP compliance. If you can write a generic test suite parameterized by the base type and have every subtype pass it, LSP is satisfied.

### 5. Documentation or Comments That Warn About Specific Subtypes

Watch for documentation, comments, or code reviews that say things like:

- "This method works for all types except..."
- "Do not pass X to this method"
- "Throws if the object is a Y"
- "Only call this after checking if the object supports it"

These are written confessions of LSP violations.

---

## 🔹 How to Fix LSP Violations

### Composition Over Inheritance

The most reliable fix. If a derived class cannot honor the base class contract, it should not be a derived class. Use composition to share code without implying substitutability:

```csharp
// Instead of Square : Rectangle, use composition
public class Square
{
    public int Side { get; set; }
    public int Area => Side * Side;

    // If you need Rectangle functionality, convert explicitly
    public Rectangle ToRectangle() => new Rectangle { Width = Side, Height = Side };
}

// Shared behavior goes in utility classes, not base classes
public static class GeometryCalculator
{
    public static double Diagonal(int width, int height)
        => Math.Sqrt(width * width + height * height);
}
```

### Interface Segregation (ISP)

When subclasses cannot support all capabilities of the base, split the capabilities into separate interfaces. This directly leverages [[4 - Interface Segregation Principle|ISP]] to prevent LSP violations:

```csharp
// Instead of one Bird class with Fly(), separate capabilities
public interface IFlyable { void Fly(); }
public interface ISwimmable { void Swim(); }
public interface IWalkable { void Walk(); }

public class Eagle : IFlyable, IWalkable
{
    public void Fly() => Console.WriteLine("Soaring.");
    public void Walk() => Console.WriteLine("Walking.");
}

public class Penguin : ISwimmable, IWalkable
{
    public void Swim() => Console.WriteLine("Swimming.");
    public void Walk() => Console.WriteLine("Waddling.");
}
```

When interfaces are slim and role-specific, implementers can honestly support every method they expose.

### Design by Contract

Explicitly document and enforce contracts. Make the contract clear so subclasses know exactly what they must honor. Use the Template Method pattern to make contract violations structurally impossible:

```csharp
public abstract class Account
{
    public decimal Balance { get; protected set; }

    // Public method is non-virtual -- contract is enforced HERE
    public void Withdraw(decimal amount)
    {
        // Precondition -- enforced by base class, cannot be bypassed
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.", nameof(amount));
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds.");

        var oldBalance = Balance;

        WithdrawCore(amount); // subclass implements the actual logic

        // Postcondition -- enforced by base class
        Debug.Assert(Balance == oldBalance - amount,
            "Postcondition violated: Balance must decrease by exactly the withdrawn amount.");
        Debug.Assert(Balance >= 0,
            "Invariant violated: Balance must never be negative.");
    }

    // Protected virtual -- subclass controls implementation, NOT the contract
    protected abstract void WithdrawCore(decimal amount);
}
```

The Template Method pattern ensures the contract is checked by the base class. Subclasses implement the behavior but cannot bypass the contract enforcement.

### "Tell, Don't Ask"

Instead of having callers check an object's type and decide what to do, let the object itself decide. Push the branching logic into the polymorphic method:

```csharp
// BAD: caller asks "what type are you?" and decides (LSP smell)
public void ProcessPayment(Account account, decimal amount)
{
    if (account is FixedDepositAccount fd)
    {
        if (DateTime.UtcNow >= fd.MaturityDate)
            fd.Withdraw(fd.Balance);
        else
            Console.WriteLine("Cannot withdraw yet.");
    }
    else
    {
        account.Withdraw(amount);
    }
}

// GOOD: tell the object what you want, let it decide how
public interface IPaymentSource
{
    PaymentResult RequestFunds(decimal amount);
}

public class SavingsPaymentSource : IPaymentSource
{
    private readonly SavingsAccount _account;
    public SavingsPaymentSource(SavingsAccount account) => _account = account;

    public PaymentResult RequestFunds(decimal amount)
    {
        if (amount > _account.Balance)
            return PaymentResult.InsufficientFunds();

        _account.Withdraw(amount);
        return PaymentResult.Success(amount);
    }
}

public class FixedDepositPaymentSource : IPaymentSource
{
    private readonly FixedDepositAccount _account;
    public FixedDepositPaymentSource(FixedDepositAccount account) => _account = account;

    public PaymentResult RequestFunds(decimal amount)
    {
        if (!_account.IsMatured)
            return PaymentResult.NotYetAvailable(_account.MaturityDate);

        if (amount != _account.Balance)
            return PaymentResult.MustWithdrawFullBalance(_account.Balance);

        _account.Redeem();
        return PaymentResult.Success(amount);
    }
}

// Caller uses polymorphism uniformly -- no type-checking
public void ProcessPayment(IPaymentSource source, decimal amount)
{
    var result = source.RequestFunds(amount);
    if (!result.IsSuccess)
        Console.WriteLine($"Payment failed: {result.Message}");
}
```

Every `IPaymentSource` implementation honestly supports `RequestFunds` -- no exceptions, no empty methods, no surprises. Each implementation handles its own business rules internally.

---

## 🔹 LSP in C#

### virtual/override Contracts

In C#, `virtual` methods establish a contract that derived classes override with `override`. The language enforces the **signature** contract (same parameters, same return type or covariant return in C# 9+), but it does **not** enforce the **behavioral** contract. It is the developer's responsibility to ensure that the override honors the base method's semantics:

```csharp
public class Logger
{
    /// <summary>
    /// Writes a message to the log.
    /// CONTRACT: must not throw for any non-null message.
    /// CONTRACT: message is written before the method returns (synchronous).
    /// </summary>
    public virtual void Log(string message)
    {
        Console.WriteLine($"[LOG] {message}");
    }
}

public class FileLogger : Logger
{
    private readonly string _filePath;
    public FileLogger(string filePath) => _filePath = filePath;

    // GOOD: honors the contract -- writes synchronously, does not throw for valid input
    public override void Log(string message)
    {
        File.AppendAllText(_filePath, $"[{DateTime.UtcNow:O}] {message}\n");
    }
}

public class LazyLogger : Logger
{
    private readonly List<string> _buffer = new();

    // VIOLATION: weakened postcondition -- message is NOT written before return.
    // It is buffered and written later (if ever). Callers that check the log
    // immediately after calling Log() will not find the message.
    public override void Log(string message)
    {
        _buffer.Add(message);
        // writes to disk later in Flush()
    }

    public void Flush()
    {
        File.AppendAllLines("app.log", _buffer);
        _buffer.Clear();
    }
}

public class FilteringLogger : Logger
{
    // VIOLATION: strengthened precondition -- silently ignores short messages
    // that the base class would have logged
    public override void Log(string message)
    {
        if (message.Length < 10)
            return; // no-op -- base class would have written this

        Console.WriteLine($"[LOG] {message}");
    }
}
```

`FileLogger` honors the contract -- it changes WHERE the message is written, but it still writes it synchronously and does not throw for valid input. Both `LazyLogger` and `FilteringLogger` violate the contract in different ways.

### sealed to Prevent Violations

When you know that a method's contract is fragile -- easy to violate by overriding incorrectly -- you can `seal` the method to prevent further overriding. This is a defensive measure to protect LSP:

```csharp
public class OrderProcessor
{
    /// <summary>
    /// Processes the order through validation, payment, and fulfillment.
    /// The sequence MUST NOT be altered by subclasses.
    /// </summary>
    public void Process(Order order) // non-virtual -- sequence is locked
    {
        Validate(order);   // step 1 -- can be customized
        Charge(order);     // step 2 -- can be customized
        Fulfill(order);    // step 3 -- can be customized
    }

    protected virtual void Validate(Order order) { /* ... */ }
    protected virtual void Charge(Order order) { /* ... */ }
    protected virtual void Fulfill(Order order) { /* ... */ }
}

public class PremiumOrderProcessor : OrderProcessor
{
    protected override void Validate(Order order)
    {
        base.Validate(order);
        // additional premium validation
    }

    // Seals Charge -- no further subclass can break the payment contract
    protected sealed override void Charge(Order order)
    {
        ApplyPremiumDiscount(order);
        base.Charge(order);
    }

    private void ApplyPremiumDiscount(Order order) { /* ... */ }
}

// This would cause a COMPILE ERROR:
public class VIPOrderProcessor : PremiumOrderProcessor
{
    // protected override void Charge(Order order) { }
    // ERROR: cannot override sealed method
}
```

You can also seal entire classes when the class should never be subclassed:

```csharp
// No subclass can ever violate this class's contract
public sealed class EmailAddress
{
    public string Value { get; }

    public EmailAddress(string value)
    {
        if (!value.Contains('@'))
            throw new ArgumentException("Invalid email address.", nameof(value));
        Value = value;
    }
}
```

> [!tip] Practical Tip
> When in doubt, seal it. Designing a class for safe inheritance is hard -- you must think about every virtual method, every postcondition, every invariant. If you do not intend for a class to be inherited, `sealed` eliminates the risk entirely. The .NET design guidelines recommend sealing classes by default.

### IReadOnlyList\<T\> vs IList\<T\> -- .NET's Own LSP Lesson

The .NET BCL's collection interfaces are a real-world case study in how LSP and [[4 - Interface Segregation Principle|ISP]] interact. `IList<T>` includes mutation methods (`Add`, `Remove`, `Insert`, etc.). `ReadOnlyCollection<T>` implements `IList<T>` but throws on every mutation method:

| Interface | Mutation Methods | LSP-Safe for Read-Only Use? |
|---|---|---|
| `IEnumerable<T>` | None | Yes |
| `IReadOnlyCollection<T>` | None | Yes |
| `IReadOnlyList<T>` | None | Yes |
| `ICollection<T>` | `Add`, `Remove`, `Clear` | No |
| `IList<T>` | `Add`, `Remove`, `Clear`, `Insert`, indexer set | No |

```csharp
// BAD: takes IList<T>, might receive ReadOnlyCollection<T>, crashes on Add
public void AppendItem<T>(IList<T> list, T item)
{
    list.Add(item); // throws NotSupportedException for ReadOnlyCollection<T>
}

// GOOD: parameter type communicates intent, LSP-safe
public void AppendItem<T>(List<T> list, T item)
{
    list.Add(item); // always works
}

// GOOD: if you only need to read, use IReadOnlyList<T>
public T GetFirstItem<T>(IReadOnlyList<T> list)
{
    return list[0]; // always works
}
```

The lesson: ==choose the parameter type that matches the contract you need==. If you need mutation, accept a type that guarantees mutation works. If you only need reading, accept a read-only type.

### NotImplementedException as a Red Flag

`NotImplementedException` in .NET means "the method is recognized but not yet implemented." `NotSupportedException` means "this operation is deliberately not supported by this type." Both are red flags for LSP violations when they appear in `override` or interface implementation methods:

| Exception | Meaning | Permanence | LSP Implication |
|---|---|---|---|
| `NotImplementedException` | "I have not implemented this yet." | Temporary -- development placeholder | Subtype is incomplete. Not yet a valid subtype. |
| `NotSupportedException` | "This operation is not supported by this type." | Permanent -- by design | Subtype cannot honor the contract. LSP violation by design. |

> [!danger] Critical Warning
> If you find `NotImplementedException` or `NotSupportedException` in an `override` method that has been in production for more than one sprint, treat it as a bug. It means the inheritance hierarchy is wrong. The fix is not to implement the method -- the fix is to restructure the type hierarchy so the method is never required on that class.

---

## 🔹 Design by Contract in C#

Design by Contract (DbC) is a software correctness methodology invented by Bertrand Meyer for the Eiffel programming language. It formalizes the contract between a method and its callers: preconditions (what the caller must ensure), postconditions (what the method guarantees), and invariants (what is always true). LSP is directly expressed in DbC terms: subtypes must not strengthen preconditions, must not weaken postconditions, and must preserve invariants.

C# does not have built-in DbC support like Eiffel, but several mechanisms exist to enforce contracts:

### Guard Clauses as Preconditions

```csharp
public virtual void Withdraw(decimal amount)
{
    // Preconditions -- runtime-enforced in all builds
    if (amount <= 0)
        throw new ArgumentException("Amount must be positive.", nameof(amount));
    if (amount > Balance)
        throw new InvalidOperationException("Insufficient funds.");

    Balance -= amount;
}

// .NET 8+ provides built-in guard methods:
public void Transfer(Account from, Account to, decimal amount)
{
    ArgumentNullException.ThrowIfNull(from);
    ArgumentNullException.ThrowIfNull(to);
    ArgumentOutOfRangeException.ThrowIfNegativeOrZero(amount);

    if (from == to)
        throw new ArgumentException("Cannot transfer to the same account.");

    from.Withdraw(amount);
    to.Deposit(amount);
}
```

Any subclass overriding `Withdraw` must NOT make the guard stricter. The base class says "any positive amount up to Balance is valid," and all subtypes must honor that.

### Debug.Assert for Postconditions and Invariants

```csharp
public virtual decimal CalculateDiscount(Order order)
{
    var discount = CalculateDiscountCore(order);

    // Postcondition: discount must be between 0% and 100%
    Debug.Assert(discount >= 0 && discount <= 1.0m,
        $"Discount {discount} is out of range [0, 1]. " +
        "Subclass violated the postcondition.");

    return discount;
}

protected virtual decimal CalculateDiscountCore(Order order)
{
    return 0.05m; // default 5% discount
}
```

`Debug.Assert` is stripped from Release builds. Use it to catch contract violations during development and testing.

### Template Method Pattern -- Strongest Contract Enforcement

The most effective way to enforce contracts on overridable methods in C# is to make the public method non-virtual and delegate to a protected virtual method, with the contract checks wrapping the call:

```csharp
public abstract class Repository<T> where T : class
{
    // Public method is non-virtual -- contract is enforced here
    public T GetById(int id)
    {
        // Precondition
        if (id <= 0)
            throw new ArgumentOutOfRangeException(nameof(id), "ID must be positive.");

        // Delegate to subclass
        var result = GetByIdCore(id);

        // Postcondition (example: could assert non-null if that's the contract)

        return result;
    }

    // Subclass implements the actual data access
    // Cannot bypass the precondition check because GetById is not virtual
    protected abstract T? GetByIdCore(int id);
}
```

This pattern guarantees that no subclass can skip precondition or postcondition checks, because those checks live in the non-virtual public method. The subclass only controls the implementation detail, not the contract boundary.

> [!tip] Practical Tip
> When designing a class hierarchy intended for extension, follow this pattern:
> 1. Make public methods **non-virtual**. These enforce the contract.
> 2. Make protected methods **virtual or abstract**. These provide the extension point.
> 3. Document the contract explicitly in XML doc comments on the public methods.
>
> This separates **contract enforcement** (base class responsibility) from **behavior customization** (subclass responsibility) and makes LSP violations structurally impossible for the contract-checked properties.

### XML Documentation as Informal Contract

```csharp
/// <summary>
/// Retrieves the user with the specified ID.
/// </summary>
/// <param name="userId">Must be a positive integer.</param>
/// <returns>Never returns null -- throws if not found.</returns>
/// <exception cref="ArgumentException">When userId is not positive.</exception>
/// <exception cref="KeyNotFoundException">When no user exists with the specified ID.</exception>
public virtual User GetUser(int userId)
{
    if (userId <= 0)
        throw new ArgumentException("User ID must be positive.", nameof(userId));

    User? user = _repository.Find(userId);
    return user ?? throw new KeyNotFoundException($"No user found with ID {userId}.");
}
```

Any subclass overriding this method must honor EVERY aspect of this documentation: accept any positive integer, never return null, throw `ArgumentException` for non-positive IDs, and throw `KeyNotFoundException` when the user does not exist. Returning null instead of throwing would weaken the postcondition and violate LSP.

### Code Contracts (Historical Context)

.NET once had `System.Diagnostics.Contracts` -- a formal DbC library with `Contract.Requires()`, `Contract.Ensures()`, and `Contract.Invariant()`. It required a separate static analyzer and binary rewriter. The project was effectively abandoned after .NET Core, but the concepts remain critically important:

```csharp
// Legacy syntax (not recommended for new projects)
public virtual void Withdraw(decimal amount)
{
    Contract.Requires(amount > 0, "Amount must be positive.");
    Contract.Requires(amount <= Balance, "Insufficient funds.");
    Contract.Ensures(Balance == Contract.OldValue(Balance) - amount);
    Contract.Ensures(Balance >= 0);

    Balance -= amount;
}
```

The modern equivalents are:
- **Guard clauses** (`if (x < 0) throw ...`) for preconditions
- **Debug.Assert** for postconditions during development
- **Roslyn analyzers** for compile-time contract checking
- **Nullable reference types** (C# 8+) for null-related postconditions
- **`ArgumentOutOfRangeException.ThrowIfNegative`** and similar (.NET 8+) for concise guards

---

## 🔹 LSP and the Other SOLID Principles

LSP does not exist in isolation. It works in concert with the other four SOLID principles, and violations of LSP often cascade into violations of the others.

**LSP + [[4 - Interface Segregation Principle|ISP]]:** Fat interfaces force implementers to provide methods they cannot support, which is both an ISP violation (forced dependency on unused methods) and an LSP violation (those methods throw or no-op). Fixing the ISP violation (splitting the interface) automatically fixes the LSP violation (each class only implements methods it can honestly deliver). ==When you find an LSP violation, check whether an ISP violation caused it. More often than not, the root cause is a fat interface.==

**LSP + [[2 - Open-Closed Principle|OCP]]:** OCP says you extend behavior through new subclasses, not by modifying existing code. But this only works if every new subclass is LSP-compliant -- if it can be substituted for the base without breaking anything. If a new subclass violates LSP, the "extension" actually breaks existing code, defeating OCP.

**LSP + [[1 - Single Responsibility Principle|SRP]]:** A class with too many responsibilities often becomes a "kitchen sink" base class that descendants override selectively, leading to empty or throwing overrides. SRP keeps base classes focused, which makes it easier for derived classes to honor the full contract.

**LSP + [[5 - Dependency Inversion Principle|DIP]]:** DIP says depend on abstractions (interfaces/abstract classes), not concretions. But this only provides value if all concrete implementations of the abstraction are LSP-compliant. If some implementations violate the contract, consuming code cannot trust the abstraction -- and DIP falls apart because you need to know which concrete type you have.

| Principle | Relationship to LSP |
|---|---|
| [[1 - Single Responsibility Principle\|SRP]] | A class with too many responsibilities is harder to subclass correctly. Each responsibility is a contract that subtypes must honor. |
| [[2 - Open-Closed Principle\|OCP]] | OCP depends on LSP. You can only safely extend via new subtypes if those subtypes are substitutable. |
| [[4 - Interface Segregation Principle\|ISP]] | ISP prevents LSP violations. Fat interfaces force subtypes to implement unsupportable methods. Slim interfaces reduce the contract surface. |
| [[5 - Dependency Inversion Principle\|DIP]] | DIP says depend on abstractions. LSP says those abstractions must be trustworthy. Without LSP, depending on abstractions is dangerous. |

---

## 🔹 LSP Violation Quick-Reference Checklist

Use this table as a rapid diagnostic tool when reviewing code:

| Symptom | What to Look For | Severity |
|---|---|---|
| `throw new NotSupportedException()` | Overridden method refuses to perform | Critical |
| `throw new NotImplementedException()` | Overridden method not yet written | High (if shipped) |
| No-op override | `override void Foo() { }` -- silently does nothing | High |
| Type checking in caller | `if (x is ConcreteType)` on base type ref | High |
| Switch on type | `switch (shape) { case Circle c: ... }` | High |
| Stronger guard clause | Subtype rejects inputs the base accepts | Critical |
| Null return where non-null expected | Weakened postcondition | High |
| Coupled property setters | Setting one property changes another | Medium-High |
| Broken invariant | State that the base guarantees is violated | Critical |
| Warning comments | "Don't call X on type Y" | Medium (confirms violation) |

---

## 🔹 Covariance and Contravariance -- LSP at the Generic Type Level

C# supports covariant (`out T`) and contravariant (`in T`) generic type parameters on interfaces and delegates. These features enforce safe substitutability at the generic type level.

**Covariance (`out T`)**: `IEnumerable<Dog>` can be used where `IEnumerable<Animal>` is expected. This is safe because `IEnumerable<T>` only *produces* `T` values (read-only). A `Dog` is always a valid `Animal`, so producing `Dog` objects where `Animal` objects are expected cannot break the contract.

**Contravariance (`in T`)**: `Action<Animal>` can be used where `Action<Dog>` is expected. This is safe because `Action<T>` only *consumes* `T` values (write-only). An action that can handle any `Animal` can certainly handle a `Dog`, since every `Dog` is an `Animal`.

```csharp
public class Animal
{
    public string Name { get; }
    public Animal(string name) => Name = name;
}

public class Dog : Animal
{
    public string Breed { get; }
    public Dog(string name, string breed) : base(name) => Breed = breed;
}

// Covariance: IEnumerable<out T>
// A list of Dogs IS-A sequence of Animals (for reading purposes)
public static void PrintAnimals(IEnumerable<Animal> animals)
{
    foreach (var animal in animals)
        Console.WriteLine(animal.Name);
}

// Contravariance: Action<in T>
// An action that handles any Animal CAN handle a Dog specifically
public static void ProcessDog(Action<Dog> action)
{
    action(new Dog("Rex", "German Shepherd"));
}

// Usage:
List<Dog> dogs = new() { new("Rex", "GSD"), new("Buddy", "Golden") };
PrintAnimals(dogs); // IEnumerable<Dog> -> IEnumerable<Animal>: covariance

Action<Animal> printAnimal = a => Console.WriteLine($"Animal: {a.Name}");
ProcessDog(printAnimal); // Action<Animal> -> Action<Dog>: contravariance
```

The compiler rejects variance declarations that would be unsafe -- you cannot declare `IList<out T>` because `IList<T>` has methods that both produce and consume `T`, and covariance would allow adding a `Cat` to a `List<Dog>`. This is the type system enforcing LSP.
