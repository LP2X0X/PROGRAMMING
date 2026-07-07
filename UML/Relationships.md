---
tags:
  - uml
  - relationships
---

## 🔹 Quick Reference Table

| Relationship     | Line Style          | Arrowhead / End      | Reads As         | Example                              |
| ---------------- | ------------------- | -------------------- | ---------------- | ------------------------------------ |
| Association       | Solid `———`         | Open arrow or none   | "knows about"    | `Customer ——— Order`                 |
| Aggregation       | Solid `———`         | Hollow diamond `◇`   | "has-a"          | `Team ◇——— Player`                   |
| Composition       | Solid `———`         | Filled diamond `◆`   | "owns"           | `Window ◆——— Button`                 |
| Dependency        | Dashed `- - -`      | Open arrow `>`       | "uses"           | `Controller - - -> Logger`           |
| Generalization    | Solid `———`         | Hollow triangle `▷`  | "is-a"           | `Dog ———▷ Animal`                    |
| Realization       | Dashed `- - -`      | Hollow triangle `▷`  | "implements"     | `SqlRepo - - -▷ IRepository`        |

> Arrow direction matters: the arrow **points toward the thing being depended on, extended, or implemented** — never toward the subclass or implementer.

---

## 🔹 Association

An association is the most general structural relationship. It means one class "knows about" another — there is some link between their instances at runtime.

### Line Notation

```
  Unidirectional (A knows B, B doesn't know A):

      Customer ————————————> Order
                  places

  Bidirectional (both know each other):

      Student <————————————> Course
                enrolled in

  Unspecified navigability (no arrows — most common in early design):

      Customer ———————————— Order
```

### Anatomy of an Association

```
          role name        association name       role name
             |                   |                   |
     +-------+     +-----------++-----------+  +-----+
     v             v                        v  v
  Customer  ——————————————————————————————————  Order
  (buyer)          places               1..*   (purchase)
                                  ^
                                  |
                            multiplicity
```

### Navigability Arrows

| Notation               | Meaning                                       |
| ---------------------- | --------------------------------------------- |
| `A ——————> B`          | A can reach B (A holds a reference to B)       |
| `A <——————> B`         | Both can reach each other                      |
| `A ———————— B`         | Navigability unspecified (decide later)         |
| `A ———x———> B`         | A cannot navigate to B's end (rare, explicit)  |

In C#, navigability almost always means "this class has a property or field of the other type":

```csharp
// Customer ——————> Order  means:
public class Customer
{
    public List<Order> Orders { get; set; }  // Customer navigates to Order
}

// Order has NO reference back to Customer
public class Order { }
```

### Association Names and Reading Direction

Write names as **verbs** and add a filled triangle `▸` to show reading direction:

```
  Customer ———— places ▸ ———— Order
  (read: "Customer places Order")

  Employee ———— ◂ works for ———— Company
  (read: "Employee works for Company")
```

### Multiplicity

Placed at each end of the association line, next to the target class:

| Notation   | Meaning                          |
| ---------- | -------------------------------- |
| `1`        | Exactly one                      |
| `0..1`     | Zero or one (optional)           |
| `*`        | Zero or more                     |
| `1..*`     | One or more                      |
| `0..*`     | Zero or more (same as `*`)       |
| `2..5`     | Between two and five             |
| `3`        | Exactly three                    |

```
  Customer  1 ———————————— 0..*  Order
  (one customer has zero or more orders)

  Order  1 ———————————— 1..*  OrderLine
  (one order has one or more order lines)
```

### Qualified Association

A qualifier narrows the "many" side down to one by using a key. Drawn as a small rectangle on the source end:

```
  ┌──────────┐ ┌────────┐        ┌───────┐
  │  Bank    │ │acctNum │————————│Account│
  │          │ │        │  0..1  │       │
  └──────────┘ └────────┘        └───────┘
                qualifier
```

This says: given a `Bank` and an `acctNum`, you get zero or one `Account`. In C# this maps to a dictionary:

```csharp
public class Bank
{
    // acctNum qualifies the association — dictionary key
    private Dictionary<string, Account> _accounts;
}
```

---

## 🔹 Aggregation (Hollow Diamond)

Aggregation is a specialized association meaning **"has-a"** where the **part can exist independently** of the whole. The hollow diamond sits on the **whole** side.

```
  Team ◇———————————— Player
  (whole)            (part)

  Department ◇———————————— Employee
  (whole)                  (part)
```

Key property: if the `Team` is dissolved, the `Player` objects still exist — they can join another team.

```csharp
public class Team
{
    // Aggregation: Team references Players, but doesn't control their lifetime
    public List<Player> Players { get; set; } = new();
}

public class Player
{
    public string Name { get; set; }
    // Player can exist without any Team
}
```

### When to Use Aggregation

- The part is **shared** or **reused** across multiple wholes
- Deleting the whole does **not** delete the parts
- The relationship is "collection-of" but not "exclusively-owns"

---

## 🔹 Composition (Filled Diamond)

Composition is a **strong** form of aggregation meaning **"owns"** — the **part cannot exist without the whole**. When the whole is destroyed, its parts are destroyed too. The filled diamond sits on the **whole** side.

```
  Window ◆———————————— Button
  (whole)              (part)

  Invoice ◆———————————— InvoiceLine
  (whole)               (part)
```

Key property: if the `Window` is destroyed, its `Button` objects are destroyed with it. A `Button` belongs to exactly one `Window`.

```csharp
public class Invoice
{
    // Composition: Invoice creates and owns its lines
    private readonly List<InvoiceLine> _lines = new();

    public void AddLine(string desc, decimal amount)
    {
        _lines.Add(new InvoiceLine(desc, amount));  // Invoice creates the part
    }
}

public class InvoiceLine
{
    // Cannot exist without an Invoice — no public constructor in strict composition
    internal InvoiceLine(string description, decimal amount) { }
}
```

### When to Use Composition

- The part has **no meaning** outside the whole
- The whole **creates and destroys** its parts
- Parts are **not shared** — each belongs to exactly one whole
- Multiplicity on the whole side is always **1** (or `0..1`)

---

## 🔹 Aggregation vs Composition — Comparison

| Criterion                     | Aggregation `◇`                     | Composition `◆`                       |
| ----------------------------- | ----------------------------------- | ------------------------------------- |
| Diamond                       | Hollow                              | Filled                                |
| Ownership                     | Weak — "has-a"                      | Strong — "owns"                       |
| Part lifetime                 | Independent of whole                | Tied to whole                         |
| Part shared?                  | Yes, can belong to multiple wholes  | No, exactly one whole                 |
| Whole deleted                 | Parts survive                       | Parts destroyed                       |
| Multiplicity (whole side)     | Any                                 | `1` (or `0..1`)                       |
| C# pattern                   | List/collection of external refs    | Private collection, whole creates     |

### Real-World C#/.NET Examples

| Scenario                                | Type          | Why                                                          |
| --------------------------------------- | ------------- | ------------------------------------------------------------ |
| `University ◇—— Professor`             | Aggregation   | Professor exists without the university                       |
| `Car ◇—— Driver`                       | Aggregation   | Driver can drive other cars, exists independently             |
| `Playlist ◇—— Song`                    | Aggregation   | Song exists in multiple playlists                             |
| `Order ◆—— OrderLine`                  | Composition   | OrderLine has no meaning without its Order                    |
| `Window ◆—— TextBox`                   | Composition   | TextBox is destroyed when its parent Window closes            |
| `House ◆—— Room`                       | Composition   | A Room doesn't exist without the House                       |
| `DbContext ◆—— ChangeTracker`          | Composition   | ChangeTracker lives and dies with its DbContext               |
| `HttpClient ◇—— HttpMessageHandler`    | Aggregation   | Handler can be shared across HttpClient instances (pooled)    |

### The "Delete Test"

> If deleting the whole **must** delete the part, it is **composition**.
> If deleting the whole **leaves the part alive**, it is **aggregation**.

---

## 🔹 Dependency (Dashed Arrow)

A dependency means one element **uses** another, but does not hold a long-lived reference. It is the **weakest** relationship. If the target changes, the source **may** be affected.

```
  Controller - - - - - - - -> Logger
  (source/client)             (target/supplier)
```

The arrow points from the **dependent** to the **thing it depends on**.

### Stereotypes

Stereotypes clarify **how** the dependency is used:

| Stereotype         | Meaning                                         | C# Example                                      |
| ------------------ | ----------------------------------------------- | ------------------------------------------------ |
| `<<use>>`          | Source uses target in a method                   | Method parameter, local variable                 |
| `<<create>>`       | Source creates instances of target               | `new SomeClass()`                                |
| `<<call>>`         | Source calls a static method on target           | `Math.Round(x)`                                  |
| `<<instantiate>>`  | Source instantiates target (template/generic)    | `Activator.CreateInstance<T>()`                   |

```
                   <<use>>
  ReportService - - - - - - - -> PdfFormatter

                  <<create>>
  OrderFactory  - - - - - - - -> Order

                   <<call>>
  Calculator    - - - - - - - -> Math
```

### Dependency in C# Code

```csharp
public class ReportService
{
    // Dependency — PdfFormatter used only as a parameter, no stored field
    public void Generate(PdfFormatter formatter)
    {
        formatter.Format(data);
    }
}
```

If `ReportService` stored `PdfFormatter` as a field, it would be an **association**, not a dependency.

### Dependency vs Association — When to Choose

| Question                                        | Association  | Dependency  |
| ----------------------------------------------- | ------------ | ----------- |
| Does the source hold a reference as a field?     | Yes          | No          |
| Is the relationship structural (long-lived)?     | Yes          | No          |
| Does the target appear only in method scope?     | No           | Yes         |

---

## 🔹 Generalization (Solid Line + Hollow Triangle)

Generalization represents **inheritance** — an "is-a" relationship. The triangle points to the **parent** (superclass / base class).

```
         Dog ————————————▷ Animal
        (child)           (parent)

  SqlRepository ————————————▷ Repository
     (child)                  (parent)
```

**Read it as**: "Dog **is a** Animal" — the arrow points from specific to general.

```
                     ┌──────────┐
                     │  Shape   │
                     └────▲─────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
        ┌─────┴──┐  ┌─────┴──┐  ┌────┴───┐
        │ Circle │  │ Square │  │Triangle│
        └────────┘  └────────┘  └────────┘
```

In C#, this is class inheritance:

```csharp
public abstract class Shape { }           // parent
public class Circle : Shape { }           // child ————▷ Shape
public class Square : Shape { }
public class Triangle : Shape { }
```

### Rules

- Arrow always points **from child to parent** (from specific to general)
- C# allows single class inheritance only — one generalization arrow per class
- Abstract classes are shown with *italic name* or `{abstract}` [[Common Notation|constraint]]

---

## 🔹 Realization (Dashed Line + Hollow Triangle)

Realization means a class **implements** an interface (or fulfills a contract). The dashed line + hollow triangle points to the **interface**.

```
  SqlRepository - - - - - - -▷ IRepository
  (implementer)                (interface)

  EmailService  - - - - - - -▷ INotificationService
  (implementer)                (interface)
```

```
                   ┌─────────────────┐
                   │ <<interface>>   │
                   │  IRepository    │
                   ├─────────────────┤
                   │ + GetById()     │
                   │ + Save()        │
                   └───────▲─────────┘
                           ╎
                   ┌───────┴─────────┐
                   │ SqlRepository   │
                   ├─────────────────┤
                   │ + GetById()     │
                   │ + Save()        │
                   └─────────────────┘
                (dashed line to interface)
```

In C#:

```csharp
public interface IRepository          // interface
{
    Entity GetById(int id);
    void Save(Entity entity);
}

public class SqlRepository : IRepository   // realization
{
    public Entity GetById(int id) { /* ... */ }
    public void Save(Entity entity) { /* ... */ }
}
```

### Generalization vs Realization

| Aspect            | Generalization `———▷`           | Realization `- - -▷`               |
| ----------------- | ------------------------------- | ---------------------------------- |
| Line style        | Solid                           | Dashed                             |
| Means             | Inherits from a class           | Implements an interface            |
| C# keyword        | `: BaseClass`                   | `: IInterface`                     |
| Target is         | Class (concrete or abstract)    | Interface                          |

---

## 🔹 ASCII Art Summary — All Relationship Types

```
  RELATIONSHIP         NOTATION                          READS AS
  ─────────────────────────────────────────────────────────────────

  Association:         A ———————————————— B               "knows"
                       A ————————————————> B               (unidirectional)

  Aggregation:         A ◇———————————————— B              "has-a"
                     (whole)              (part)

  Composition:         A ◆———————————————— B              "owns"
                     (whole)              (part)

  Dependency:          A - - - - - - - -> B               "uses"
                    (client)           (supplier)

  Generalization:      A ————————————————▷ B              "is-a"
                     (child)            (parent)

  Realization:         A - - - - - - - -▷ B              "implements"
                   (implementer)      (interface)
```

### Diamond and Arrow Placement Rule

```
  ◇ or ◆ always goes on the WHOLE side (the container, the owner)
  ▷       always points TOWARD the parent or interface
  >       always points TOWARD the thing being depended on / navigated to
```

---

## 🔹 Common Mistakes

### 1. Confusing Aggregation and Composition

**Wrong**: Using `◆` (composition) for `University ◆—— Professor` — a professor survives if the university closes.

**Right**: `University ◇—— Professor` (aggregation). Apply the delete test.

### 2. Reversed Generalization Arrow

**Wrong**:
```
  Animal ————————————▷ Dog
  (this says Animal is-a Dog!)
```

**Right**:
```
  Dog ————————————▷ Animal
  (Dog is-a Animal — arrow from child to parent)
```

The triangle always points to the **more general** thing. Think: the arrow goes "up" the hierarchy.

### 3. Association When It Should Be Dependency

**Wrong**: Drawing a solid association between `Controller` and `Logger` when `Logger` is only used as a method parameter.

**Right**: Use a dashed dependency arrow. If there is no stored field/property, it is a dependency, not an association.

### 4. Omitting Multiplicity

Multiplicity is not optional decoration — it conveys critical design constraints. `1..*` vs `0..*` determines whether null checks or validation are needed. Always specify it on associations, aggregations, and compositions.

### 5. Using Aggregation Everywhere

The UML spec itself notes that aggregation semantics are "not precisely defined." When in doubt between plain association and aggregation, **prefer plain association**. Only use `◇` when the whole-part semantics are genuinely meaningful to the domain.

### 6. Confusing Generalization and Realization

**Wrong**: Solid line + hollow triangle to an `<<interface>>`.

**Right**: Interfaces always get **dashed** lines (realization). Solid lines are for class inheritance (generalization).

---

## 🔹 Decision Guide — "Which Relationship Do I Use?"

Follow these questions in order:

```
  START
    │
    ▼
  Does class A inherit from class B?
    │
    ├─ YES, B is a class ──────────────> GENERALIZATION  A ———▷ B
    │
    ├─ YES, B is an interface ─────────> REALIZATION     A - - ▷ B
    │
    └─ NO
       │
       ▼
     Does A hold a reference to B (field / property)?
       │
       ├─ NO ──> Does A use B temporarily
       │         (parameter, local, return type)?
       │           │
       │           ├─ YES ─────────────> DEPENDENCY      A - - -> B
       │           │
       │           └─ NO ─────────────> (no relationship)
       │
       └─ YES
          │
          ▼
        Is B a "part" of A (whole-part semantics)?
          │
          ├─ NO ───────────────────────> ASSOCIATION      A ——— B
          │
          └─ YES
             │
             ▼
           Can B exist without A?
           (Apply the delete test)
             │
             ├─ YES ──────────────────> AGGREGATION      A ◇——— B
             │
             └─ NO ───────────────────> COMPOSITION      A ◆——— B
```

### Quick Decision Table

| Situation in C# Code                                     | Relationship    |
| -------------------------------------------------------- | --------------- |
| `class Dog : Animal`                                     | Generalization  |
| `class SqlRepo : IRepository`                            | Realization     |
| `class A { private B _b; }`                              | Association     |
| `class A { private List<B> _parts; }` (A owns B's life) | Composition     |
| `class A { public List<B> Items; }` (B exists alone)     | Aggregation     |
| `void Method(B b)` — B only in method scope              | Dependency      |
| `var b = new B();` inside a method, no field             | Dependency      |

---

See also: [[Common Notation]], [[Class Diagrams]], [[Multiplicity]], [[Interfaces and Abstract Classes]]
