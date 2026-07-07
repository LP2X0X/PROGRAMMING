---
tags:
 - csharp
 - oop
 - functional-programming
 - paradigms
 - linq
aliases:
 - OOP vs FP
 - Object-Oriented vs Functional
 - Paradigm Comparison
status: complete
---

# OOP vs Functional Programming

> [!ad-note] Overview
> Object-oriented programming and functional programming are two fundamentally different ways of thinking about code. OOP models the world as **things that do stuff** (nouns). FP models the world as **transformations that happen to data** (verbs). Modern C# blends both paradigms — you already use functional patterns daily through LINQ, lambdas, and records without necessarily labeling them as "functional." This note breaks down the philosophical difference, shows the same problem solved both ways in C#, and gives practical guidance on when to lean toward each style.

---

## Table of Contents

- [The Core Philosophical Split](#the-core-philosophical-split)
- [Same Problem, Two Approaches](#same-problem-two-approaches)
- [The Four Key Differences](#the-four-key-differences)
- [You Already Use Functional Patterns](#you-already-use-functional-patterns)
- [The Real Tradeoff: Resources vs Bugs](#the-real-tradeoff-resources-vs-bugs)
- [When to Use Which](#when-to-use-which)
- [C# Features by Paradigm](#c-features-by-paradigm)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## The Core Philosophical Split

The difference between OOP and FP is not about syntax or language features. It is about ==how you model the world==.

### OOP: The World Is Made of Things

- OOP asks: **"What THINGS exist, and what can they DO?"**
- You model the world as ==nouns== — objects that have identity, carry state, and expose behavior
- An `Order` is a *thing*. It knows its items, its total, its status. It can `Validate()` itself, `CalculateTotal()`, and `Save()` itself to a database
- The object **owns** its data and **controls** who can touch it ([[Role of Encapsulation]])
- Objects interact by sending messages to each other (method calls)

```csharp
// OOP worldview: Order is a thing that does stuff
var order = new Order(items);
order.Validate();        // the order validates itself
order.CalculateTotal();  // the order calculates its own total
order.Save();            // the order saves itself
```

The mental model is: **I have an Order object. I tell it what to do. It manages its own state.**

### Functional: The World Is Made of Transformations

- FP asks: **"What TRANSFORMATIONS happen to the data?"**
- You model the world as ==verbs== — functions that take data in and produce new data out
- An `Order` is just *data* — a bag of values with no behavior attached. Separate functions operate on that data
- The data does not know or care what happens to it. Functions do all the work
- Each function takes input, returns output, and ==never modifies the original==

```csharp
// FP worldview: Order is just data, functions do the work
var order = new Order(items);
var validated = OrderValidator.Validate(order);    // returns new validated order
var totaled = OrderCalculator.CalculateTotal(validated); // returns new order with total
OrderRepository.Save(totaled);                     // takes data, writes it
```

The mental model is: **I have data. I pass it through a pipeline of functions. Each step produces new data.**

> [!ad-note] Neither Is "Better"
> These are lenses, not religions. The same real-world problem can be modeled either way. The question is which lens gives you clearer, safer, more maintainable code *for your specific situation*.

> [!ad-note] Section Summary
> - OOP models the world as ==nouns== (objects with state + behavior)
> - FP models the world as ==verbs== (functions that transform data)
> - OOP: objects own and mutate their own state
> - FP: data is inert, functions produce new data without modifying the original
> - Both are valid mental models — the right choice depends on the problem

---

## Same Problem, Two Approaches

Let's process an order end-to-end in both styles. This makes the difference concrete.

### The OOP Way: Object Owns Everything

```csharp
public class Order
{
    public List<OrderItem> Items { get; private set; }
    public decimal Total { get; private set; }
    public bool IsValid { get; private set; }
    public OrderStatus Status { get; private set; }

    public Order(List<OrderItem> items)
    {
        Items = items;
        Status = OrderStatus.Created;
    }

    // Behavior lives INSIDE the object
    public void Validate()
    {
        if (Items == null || Items.Count == 0)
            throw new InvalidOperationException("Order must have items.");
        
        IsValid = true;
        Status = OrderStatus.Validated;  // object MUTATES itself
    }

    public void CalculateTotal()
    {
        if (!IsValid)
            throw new InvalidOperationException("Validate first.");
        
        Total = Items.Sum(i => i.Price * i.Quantity);
        Total *= 1.08m; // tax
        Status = OrderStatus.Calculated;  // another mutation
    }

    public void Save(IDatabase db)
    {
        if (Status != OrderStatus.Calculated)
            throw new InvalidOperationException("Calculate total first.");
        
        db.Insert(this);
        Status = OrderStatus.Saved;  // yet another mutation
    }
}
```

**Usage:**

```csharp
var order = new Order(items);
order.Validate();
order.CalculateTotal();
order.Save(db);

// The SAME object has been mutated three times.
// order.Status went: Created → Validated → Calculated → Saved
```

Notice:
- The `Order` object holds state (`Total`, `IsValid`, `Status`) **and** behavior (`Validate()`, `CalculateTotal()`, `Save()`)
- Each method **mutates** the object in place — it changes `this`
- Method call order matters — calling `CalculateTotal()` before `Validate()` throws
- The object has a **lifecycle** with state transitions

### The Functional Way: Data + Standalone Functions

```csharp
// Data is just a record — no behavior, immutable by default
public record OrderData(
    ImmutableList<OrderItem> Items,
    decimal Total = 0m,
    bool IsValid = false,
    OrderStatus Status = OrderStatus.Created
);

// Functions are standalone — they take data in, return NEW data out
public static class OrderPipeline
{
    public static OrderData Validate(OrderData order)
    {
        if (order.Items.IsEmpty)
            throw new ArgumentException("Order must have items.");
        
        // Returns a NEW order — original is untouched
        return order with { IsValid = true, Status = OrderStatus.Validated };
    }

    public static OrderData CalculateTotal(OrderData order)
    {
        var total = order.Items.Sum(i => i.Price * i.Quantity) * 1.08m;
        
        // Another NEW order
        return order with { Total = total, Status = OrderStatus.Calculated };
    }

    public static OrderData Save(OrderData order, IDatabase db)
    {
        db.Insert(order);
        return order with { Status = OrderStatus.Saved };
    }
}
```

**Usage:**

```csharp
var order = new OrderData(items);
var validated = OrderPipeline.Validate(order);
var calculated = OrderPipeline.CalculateTotal(validated);
var saved = OrderPipeline.Save(calculated, db);

// We now have FOUR separate objects:
// order      → Status: Created     (unchanged!)
// validated  → Status: Validated
// calculated → Status: Calculated
// saved      → Status: Saved
```

Notice:
- `OrderData` is a [[Record]] — it holds data and nothing else
- Each function takes an `OrderData` in and returns a **new** `OrderData` out
- The original `order` is ==never modified== — you can still inspect it
- Functions are **standalone** — they don't live inside the data class
- You get a complete history: every intermediate state is preserved

> [!ad-tip] The `with` Expression
> The `with` keyword on records is what makes the functional style practical in C#. It creates a shallow copy with only the specified properties changed. Without it, you would need to manually construct new objects every time — which is why FP was historically painful in C# before records were introduced in C# 9.

> [!ad-note] Section Summary
> - OOP: the `Order` object holds state and behavior together, mutating itself through a lifecycle
> - FP: `OrderData` is pure data (a record), standalone functions transform it into new copies
> - OOP mutates in place (one object, many state changes); FP creates new copies (many objects, no mutations)
> - The `with` expression on records makes the functional pattern ergonomic in C#
> - Both approaches solve the same problem — the difference is where state changes happen and who is responsible

---

## The Four Key Differences

| Dimension | OOP | Functional |
|---|---|---|
| **State** | Objects ==mutate themselves==. `order.Status = Saved;` changes the existing object. | Data is ==immutable==. Functions return new copies: `order with { Status = Saved }`. |
| **Behavior** | Methods live **inside** the class. `order.Validate()` — the object does the work. | Functions are **standalone**. `Validate(order)` — the function does the work on the data. |
| **Identity** | An object **is something**. Two `Order` objects with the same data are still different objects (reference identity). | Data **is just values**. Two records with the same data are equal (`==` returns true). |
| **Composition** | Build complexity through **inheritance** and **interfaces**. `PremiumOrder : Order`. | Build complexity by **chaining functions**. `Validate → CalculateTotal → ApplyDiscount → Save`. |

### Expanding on Each

**State** — This is the most consequential difference. In OOP, when you call `order.Validate()`, the original order object is *gone* — it has been mutated into a validated order. In FP, you still have the original unvalidated order *and* the new validated one. This has enormous implications for debugging, testing, and concurrency.

**Behavior** — In OOP, you ask "what can this object *do*?" and look at its methods. In FP, you ask "what functions can I *apply* to this data?" and look at the available functions. OOP groups code by *what it belongs to*. FP groups code by *what it does*.

**Identity** — In OOP, `new Order(items) != new Order(items)` even if the data is identical (they are different references on the heap). In FP with records, `new OrderData(items) == new OrderData(items)` because identity is defined by the values. See [[Object Identity vs Object Equality]].

**Composition** — OOP's primary tool for code reuse is inheritance: `class PremiumOrder : Order`. FP's primary tool is function composition: pipe data through a chain of small, focused functions. LINQ's `.Where().Select().OrderBy()` is a textbook example of function composition.

> [!ad-warning] Common Misconception
> "Functional programming means no classes." **Wrong.** C# records are classes (or structs). You absolutely use types to define your data shapes in FP. The difference is that in FP, those types carry *data only* — no methods that mutate state. The behavior lives in separate functions.

> [!ad-note] Section Summary
> - **State**: OOP mutates objects in place; FP returns new immutable copies
> - **Behavior**: OOP puts methods inside classes; FP keeps functions standalone
> - **Identity**: OOP identity is reference-based; FP identity is value-based
> - **Composition**: OOP composes via inheritance/interfaces; FP composes via function chaining
> - The state difference (mutable vs immutable) is the most impactful in practice

---

## You Already Use Functional Patterns

Here is the key insight: ==if you write C#, you are already doing functional programming==. You just might not be labeling it that way.

### LINQ Is Functional

Every time you write a LINQ chain, you are doing textbook functional programming:

```csharp
var expensiveItems = orders
    .Where(o => o.Total > 100)       // pure function: takes order, returns bool
    .SelectMany(o => o.Items)         // pure function: takes order, returns items
    .Where(i => i.Price > 50)         // another pure function
    .OrderByDescending(i => i.Price)  // another pure function
    .ToList();
```

This is a ==pipeline of pure functions==. Data flows through. No object is mutated. Each step takes input and produces output. The original `orders` collection is untouched.

This is *exactly* what functional programming is.

### Lambdas Are Functions as Values

```csharp
// You're passing a FUNCTION as a value — a core FP concept
Func<decimal, bool> isExpensive = price => price > 100;

items.Where(isExpensive);          // function passed as argument
items.Where(i => i.Price > 100);   // anonymous function inline
```

In OOP, behavior is attached to objects. In FP, ==functions are values== — you can store them in variables, pass them as arguments, return them from other functions. Every [[Lambda Expression Overview|lambda expression]] you write is an FP concept. Every [[Callback]] you pass is FP.

### Records Are Immutable Data

```csharp
record Product(string Name, decimal Price);

var original = new Product("Widget", 9.99m);
var updated = original with { Price = 12.99m };

// original is still { Name = "Widget", Price = 9.99 }
// updated  is      { Name = "Widget", Price = 12.99 }
```

A [[Record]] is pure data with no behavior. The `with` expression creates a new copy instead of mutating. This is the FP data model.

### Pattern Matching Is an FP Staple

```csharp
// Instead of polymorphism (OOP), use pattern matching (FP):
string Describe(Shape shape) => shape switch
{
    Circle c    => $"Circle with radius {c.Radius}",
    Rectangle r => $"Rectangle {r.Width}x{r.Height}",
    _           => "Unknown shape"
};
```

In OOP, you would make `Shape` an abstract class with a virtual `Describe()` method, and each subclass overrides it. In FP, the data types are simple and the behavior is external — pattern matching on the data's *shape* replaces virtual dispatch.

### ASP.NET Core Middleware Is Function Composition

```csharp
app.UseAuthentication();
app.UseAuthorization();
app.UseRateLimiting();
app.MapControllers();
```

The [[Middleware Overview|middleware pipeline]] is a chain of functions. Each takes a request, does something, and passes it to the next function. This is function composition — the same idea as LINQ, just applied to HTTP requests instead of collections.

### Minimal APIs Are Lambdas Mapped to Routes

```csharp
app.MapGet("/products", (ProductDb db) =>
    db.Products
      .Where(p => p.IsActive)
      .OrderBy(p => p.Name)
      .ToList());
```

[[Minimal APIs]] throw away the OOP ceremony entirely. No controller class, no constructor injection, no `[HttpGet]` attributes. Just a lambda (function) mapped directly to a route. This is why they "feel functional" — because they *are* functional.

> [!ad-tip] The "Aha" Moment
> If LINQ, lambdas, records, pattern matching, middleware, and Minimal APIs all feel natural to you — congratulations, you already think functionally when the situation calls for it. You just didn't have a name for it. The key insight is that C# is a **multi-paradigm** language. You don't have to choose one approach for everything.

> [!ad-note] Section Summary
> - LINQ is a pure functional pipeline — data in, data out, no mutations
> - Lambdas are functions treated as values — a core FP concept
> - Records are immutable data carriers — the FP data model
> - Pattern matching replaces virtual dispatch with external data-shape analysis
> - ASP.NET Core middleware and Minimal APIs are function composition applied to web requests
> - You are already doing FP daily — C# is multi-paradigm

---

## The Real Tradeoff: Resources vs Bugs

When you first learn about immutability, a natural reaction is: *"Creating new copies every time? Isn't that wasteful?"*

It is a fair question. Let's address it head-on.

### Yes, Copies Cost Memory

```csharp
// FP style: three allocations instead of one
var order = new OrderData(items);
var validated = OrderPipeline.Validate(order);      // new object
var calculated = OrderPipeline.CalculateTotal(validated); // another new object
```

In the OOP version, you mutated one object. In the FP version, you created three. That is a real cost.

### But C# Mitigates This

- **LINQ is lazy.** `.Where().Select().OrderBy()` does not create intermediate collections. It builds an iterator chain. Only `.ToList()` materializes the result
- **Records share references.** `order with { Status = Saved }` copies the record's fields, but the `Items` list is *the same reference* — it is not deep-cloned. For a record with a few value-type fields, a `with` copy is just a struct-copy-sized allocation
- **The JIT optimizes aggressively.** Short-lived objects in Gen 0 are collected cheaply. The GC is designed for this pattern
- **`Span<T>` and stack allocation** give you zero-copy slicing for performance-critical paths where even FP-style immutability is too expensive

### The Real Tradeoff: Memory vs Bugs

==The question is not "does immutability cost more memory?" — it does. The question is "is that cost worth the bugs it prevents?"==

Consider this OOP code in a multi-threaded environment:

```csharp
// Shared mutable state — the classic disaster
public class OrderProcessor
{
    private decimal _runningTotal = 0; // shared mutable state

    public void ProcessOrder(Order order)
    {
        _runningTotal += order.Total; // NOT thread-safe
        
        if (_runningTotal > 10_000)
        {
            ApplyBulkDiscount();      // race condition here
            _runningTotal = 0;
        }
    }
}
```

Two threads calling `ProcessOrder` simultaneously:

```
Thread A reads _runningTotal = 9,500
Thread B reads _runningTotal = 9,500        ← stale value!
Thread A adds 600  → writes 10,100
Thread A enters if block, applies discount
Thread B adds 700  → writes 10,200          ← overwrites A's write!
Thread B enters if block, applies discount  ← discount applied TWICE
Thread A resets to 0
Thread B resets to 0                        ← both reset, data lost
```

This is a **race condition** caused by ==shared mutable state==. Two threads reading and writing the same variable without coordination. The result is corrupted data, double-applied discounts, and bugs that only appear under load — the worst kind.

### Immutability Makes This Impossible

```csharp
// Immutable version — no shared mutable state exists
public static class OrderProcessor
{
    public static ProcessingResult ProcessOrder(
        ProcessingState state, OrderData order)
    {
        var newTotal = state.RunningTotal + order.Total;
        
        if (newTotal > 10_000)
        {
            return new ProcessingResult(
                NewState: state with { RunningTotal = 0 },
                ApplyDiscount: true);
        }
        
        return new ProcessingResult(
            NewState: state with { RunningTotal = newTotal },
            ApplyDiscount: false);
    }
}
```

Each thread works on its own copy of state. No thread can see or corrupt another thread's data. The race condition is ==structurally impossible== — not prevented by locks, but eliminated by design.

> [!ad-warning] The WinForms Connection
> If you have done WinForms development, you know this error:
> ```
> System.InvalidOperationException: 
> Cross-thread operation not valid: Control 'textBox1' accessed from a 
> thread other than the thread it was created on.
> ```
> This is the **same underlying problem** — shared mutable state (the UI control) being accessed from multiple threads. WinForms "solves" it by throwing an exception. FP solves it by making the situation impossible in the first place.

### The Cost-Benefit in Context

In a desktop app processing one thing at a time, mutable OOP is fine. The concurrency risk is low.

In a web server handling thousands of concurrent requests — each one a separate thread, all potentially accessing shared services — ==immutability is not a luxury, it is a survival strategy==.

A few extra Gen 0 allocations are measured in nanoseconds. A race condition bug in production is measured in hours of debugging and lost revenue.

> [!ad-note] Section Summary
> - Yes, creating immutable copies uses more memory than mutating in place
> - C# mitigates this: LINQ is lazy, records share references, the GC handles short-lived objects efficiently
> - The real tradeoff is a few extra allocations vs entire categories of eliminated bugs
> - Shared mutable state causes race conditions — bugs that only appear under load and are extremely hard to reproduce
> - Immutability makes race conditions structurally impossible, not just prevented
> - In concurrent environments (web servers, async code), immutability pays for itself many times over

---

## When to Use Which

This is not an either/or decision. ==Modern C# is multi-paradigm==. The skill is knowing which tool to reach for.

### Use OOP When...

| Scenario | Why OOP Fits |
|---|---|
| **Structuring large systems** | Classes, interfaces, and DI give you architectural boundaries. A `UserService` that depends on `IUserRepository` is OOP composition through interfaces. |
| **Domain modeling** | Complex business entities with rules and lifecycle make sense as objects. An `Account` that enforces "cannot withdraw more than balance" is natural OOP. |
| **Stateful long-lived objects** | A `DatabaseConnection`, a `FileStream`, a WinForms `Form` — these are inherently stateful. Modeling them as immutable data would be awkward. |
| **Hot performance-critical paths** | When you genuinely cannot afford allocations, mutating a pre-allocated buffer in place (OOP-style) beats creating new copies. |

### Use Functional When...

| Scenario | Why FP Fits |
|---|---|
| **Transforming data** | Processing a list of orders, filtering products, mapping DTOs. LINQ pipelines are FP, and they are the right tool. |
| **Stateless request handling** | Each HTTP request in ASP.NET Core should be handled without shared mutable state. Middleware and Minimal API handlers are naturally functional. |
| **Business logic where correctness matters** | Calculations, validations, rule evaluation. Pure functions (same input always gives same output) are trivially testable. |
| **Concurrent / async code** | When multiple threads or async tasks run simultaneously, immutable data eliminates an entire class of synchronization bugs. |

### The Blend in Practice

Real-world C# code blends both paradigms constantly:

```csharp
// OOP: class with DI, encapsulation, interface implementation
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repo;
    
    public OrderService(IOrderRepository repo) // DI — OOP pattern
    {
        _repo = repo;
    }

    public async Task<OrderSummary> GetTopOrdersAsync(int count)
    {
        var orders = await _repo.GetAllAsync();

        // Functional: LINQ pipeline, immutable transformations
        var topOrders = orders
            .Where(o => o.Status == OrderStatus.Completed)
            .OrderByDescending(o => o.Total)
            .Take(count)
            .Select(o => new OrderSummary(o.Id, o.Total, o.Date))
            .ToList();

        return topOrders;
    }
}
```

The *structure* is OOP — a class, an interface, constructor injection, encapsulation. The *data processing* inside is functional — a LINQ pipeline that transforms data without mutation. Neither paradigm owns the whole picture.

> [!ad-tip] Rule of Thumb
> **OOP for the architecture, FP for the logic.** Use classes, interfaces, and DI to wire your system together. Use LINQ, records, and pure functions to process data within those classes. This is idiomatic modern C#.

> [!ad-note] Section Summary
> - Use OOP for system structure: classes, interfaces, DI, domain entities, stateful objects
> - Use FP for data processing: LINQ pipelines, validations, calculations, stateless transformations
> - For hot paths where allocation matters, lean OOP (mutate in place)
> - For business logic where correctness matters, lean FP (immutable, testable)
> - Idiomatic modern C# blends both — OOP architecture with FP data processing inside

---

## C# Features by Paradigm

| Feature | Paradigm | Notes |
|---|---|---|
| Classes | OOP | The fundamental OOP building block. State + behavior bundled together. |
| Inheritance | OOP | `class Dog : Animal` — code reuse through "is-a" relationships. |
| Interfaces | OOP | [[Role of Abstraction]] through contracts. Enables DI and polymorphism. |
| `virtual` / `override` | OOP | Runtime polymorphism — the object decides which method runs. |
| Properties | OOP | [[Role of Encapsulation]] — controlled access to internal state. |
| Encapsulation (`private`, `protected`) | OOP | Hiding internal state from external code. |
| Lambdas | Functional | [[Lambda Expression Overview]] — functions as first-class values. |
| LINQ | Functional | Declarative data transformation pipelines. Pure FP in C# syntax. |
| Records | Functional | [[Record]] — immutable value-based data types. The FP data model. |
| Pattern matching | Functional | External behavior based on data shape, replacing virtual dispatch. |
| Tuples | Functional | Lightweight composite return values without defining a class. |
| Expression-bodied members | Functional | `int Double(int x) => x * 2;` — concise function definitions. |
| `Span<T>` | Functional | Zero-copy slicing of contiguous memory. Functional in spirit (no mutation of source). |
| Generics | Both | Used by OOP (`List<T>`, `IRepository<T>`) and FP (`Func<T, TResult>`, LINQ) equally. |
| `async` / `await` | Both | OOP services use it for I/O. FP chains use it in async LINQ. Neither paradigm owns it. |
| Extension methods | Both | OOP uses them to extend types. FP uses them to build fluent pipelines (LINQ is extension methods). |
| [[Callback\|Delegates]] | Both | OOP uses delegates for events. FP uses them as function types (`Func<>`, `Action<>`). |

> [!ad-warning] Common Misconception
> "Functional programming means no state at all." **Wrong.** FP absolutely has state — it just does not *mutate* state. You create new state by returning new values from functions. The database still gets written to. The HTTP response still gets sent. The difference is that your in-memory data is not modified in place during processing.

> [!ad-note] Section Summary
> - C# has dedicated features for both paradigms and features that serve both
> - OOP features focus on structure, identity, and encapsulation (classes, inheritance, interfaces)
> - FP features focus on transformation, values, and composition (lambdas, LINQ, records, pattern matching)
> - Shared features (generics, async/await, extension methods) are paradigm-neutral tools
> - FP does not mean "no state" — it means state flows through functions instead of being mutated in place

---

## Comprehensive Summary

> [!ad-tip] Complete Summary
> OOP and FP are two ways of thinking about code. **OOP models the world as nouns** — objects that hold state and expose behavior. You tell an object to do things (`order.Validate()`), and it modifies itself. **FP models the world as verbs** — standalone functions that transform immutable data. You pass data through functions (`Validate(order)`), and each function returns new data without touching the original.
>
> The four key differences are: (1) **State** — OOP mutates objects in place, FP creates new immutable copies; (2) **Behavior** — OOP methods live inside classes, FP functions are standalone; (3) **Identity** — OOP objects have reference identity, FP data has value identity; (4) **Composition** — OOP uses inheritance/interfaces, FP chains functions together.
>
> If you write C#, you already use FP constantly: LINQ pipelines are function composition, lambdas are functions as values, records are immutable data, pattern matching is external behavior dispatch, and Minimal APIs and middleware are function composition applied to web requests.
>
> The practical tradeoff between paradigms is **memory vs bugs**. Immutable copies cost more memory than mutations, but C# mitigates this (lazy LINQ, reference sharing, efficient GC). The bigger win is that immutability eliminates race conditions and shared-mutable-state bugs entirely — not by prevention (locks), but by design (the problematic state simply does not exist). In concurrent environments like web servers, this matters far more than a few extra allocations.
>
> The expert move is to blend both: **use OOP for system architecture** (classes, interfaces, DI, domain models, stateful resources) and **FP for data processing** (LINQ pipelines, validations, calculations, stateless transformations). This is idiomatic modern C# — a multi-paradigm language where knowing *when* to reach for each style is what separates competent code from excellent code.

---

## Related Topics

- [[Lambda Expression Overview]] — Functions as first-class values in C#
- [[Record]] — Immutable, value-based data types (the FP data model in C#)
- [[Role of Encapsulation]] — OOP's mechanism for protecting internal state
- [[Role of Abstraction]] — OOP's mechanism for hiding complexity behind contracts
- [[Minimal APIs]] — The functional style of defining ASP.NET Core endpoints
- [[Middleware Overview]] — Function composition applied to the HTTP pipeline
- [[Callback]] — Passing functions as arguments — FP in action
- [[Object Identity vs Object Equality]] — Reference vs value identity (OOP vs FP worldview)
- [[Class vs Abstract Class vs Interface]] — OOP composition tools
