---
tags:
  - uml
  - notation
  - syntax
---

## 🔹 Universal Symbol Quick Reference

| Symbol | Meaning | Example |
|--------|---------|---------|
| `+` `-` `#` `~` | Visibility (public, private, protected, package) | `+ getName(): String` |
| `<<stereotype>>` | Classifier category / role | `<<interface>>`, `<<abstract>>` |
| `{constraint}` | Constraint or tagged value | `{readOnly}`, `{ordered}` |
| `:` | Type separator | `name : String` |
| `=` | Default value | `count : int = 0` |
| `[ ]` | Multiplicity | `items [0..*]` |
| `...` | Elision (more members exist) | Listed at end of compartment |
| `/` | Derived element | `/age : int` (computed, not stored) |
| `_underline_` | Static (class-level) member | `_+ getInstance()_` |
| *italic* | Abstract element | *calculateArea()* |
| `::` | Namespace qualifier | `Shapes::Circle` |
| `--` | Separator between compartments | Between attributes and operations |
| `{ }` | Tagged values / properties | `{unique, ordered}` |
| `(* *)` | Comment delimiters inside notes | `(* invariant *)` |

## 🔹 Visibility

Visibility controls access to attributes, operations, and other members. It appears as a single prefix character before the member name.

| Symbol | Keyword | Who Can Access | When to Use |
|--------|---------|----------------|-------------|
| `+` | public | Any classifier | API surface, interface methods |
| `-` | private | Only the owning classifier | Internal state, helper methods |
| `#` | protected | Owning classifier + subclasses | Extension points for inheritance |
| `~` | package | Classifiers in the same package | Internal collaboration within a module |

**In diagrams:**

```
+---------------------------------------+
|           BankAccount                  |
+---------------------------------------+
| - balance : Decimal                    |
| # accountId : String                   |
| ~ auditLog : List<String>             |
+---------------------------------------+
| + deposit(amount : Decimal) : void     |
| + withdraw(amount : Decimal) : Boolean |
| - validateAmount(amt : Decimal) : void |
| # notifyObservers() : void             |
+---------------------------------------+
```

- `balance` is `-` private: only `BankAccount` itself can touch raw balance.
- `accountId` is `#` protected: subclasses like `SavingsAccount` inherit access.
- `auditLog` is `~` package: other classes in the banking package can read it.
- `deposit()` and `withdraw()` are `+` public: the external API.
- `validateAmount()` is `-` private: internal guard, callers never see it.
- `notifyObservers()` is `#` protected: subclasses can override notification behavior.

Visibility applies to [[Relationships]] too -- association ends can carry visibility markers to show navigability restrictions.

## 🔹 Stereotypes

Stereotypes extend the UML vocabulary. They appear in guillemets `<< >>` above or before the classifier name.

### Built-in Stereotypes

| Stereotype | Applied To | Meaning |
|------------|-----------|---------|
| `<<interface>>` | Class | No implementation; pure contract. All operations are abstract. |
| `<<abstract>>` | Class / Operation | Cannot be instantiated directly / must be overridden. |
| `<<enumeration>>` | Class | Fixed set of named values (literals). |
| `<<utility>>` | Class | All members are static; no instances created. |
| `<<dataType>>` | Class | Value type with no identity (like a struct). |
| `<<primitive>>` | Class | Built-in primitive (`int`, `boolean`, `String`). |
| `<<signal>>` | Class | Asynchronous notification object. |
| `<<exception>>` | Class | Throwable error type. |
| `<<actor>>` | Class | External entity interacting with the system ([[Use Case Diagram]]). |
| `<<boundary>>` | Class | UI or system boundary (Robustness / [[Sequence Diagram]]). |
| `<<control>>` | Class | Coordination logic (Robustness analysis). |
| `<<entity>>` | Class | Persistent domain object (Robustness analysis). |

### Stereotype Placement in Diagrams

```
+-----------------------------+       +-----------------------------+
|     <<interface>>           |       |      <<enumeration>>        |
|        Drawable             |       |         Color               |
+-----------------------------+       +-----------------------------+
| + draw(ctx : Context): void |       |  RED                        |
| + resize(factor : double)   |       |  GREEN                      |
+-----------------------------+       |  BLUE                        |
                                      +-----------------------------+

+-----------------------------+
|      <<utility>>            |
|       MathHelper            |
+-----------------------------+
| + PI : double = 3.14159     |
+-----------------------------+
| + sqrt(x : double) : double |
| + abs(x : double) : double  |
+-----------------------------+
```

The stereotype always sits in the **top compartment**, above the class name. When a class has both a stereotype and a name, the stereotype goes on the line above.

### Custom Stereotypes

You can define your own stereotypes for domain-specific modeling:

| Custom Stereotype | Typical Use |
|-------------------|-------------|
| `<<service>>` | Application/domain service class |
| `<<repository>>` | Data access layer class |
| `<<factory>>` | Object creation responsibility |
| `<<DTO>>` | Data transfer object (no behavior) |
| `<<valueObject>>` | DDD value object (identity by value) |
| `<<aggregate>>` | DDD aggregate root |
| `<<event>>` | Domain event |

Custom stereotypes can also carry tagged values and constraints, forming a **profile** -- a lightweight extension mechanism without changing the UML metamodel.

## 🔹 Tagged Values (Properties)

Tagged values are name-value pairs in curly braces `{ }` that attach metadata to any model element.

### Standard Tagged Values

| Tagged Value | Applies To | Meaning |
|-------------|-----------|---------|
| `{readOnly}` | Attribute, Association end | Value cannot be changed after initialization |
| `{ordered}` | Association end, Attribute | Elements maintain insertion order |
| `{unique}` | Association end, Attribute | No duplicate elements allowed |
| `{nonunique}` | Association end | Duplicates are permitted |
| `{subsets X}` | Association end | This collection is a subset of property X |
| `{redefines X}` | Association end | Overrides inherited property X |
| `{union}` | Association end | Derived union of all subsets |
| `{composite}` | Association end | Equivalent to filled diamond ([[Relationships#Composition]]) |
| `{abstract}` | Operation | Must be implemented by subclasses |
| `{query}` | Operation | Does not modify the object state |
| `{leaf}` | Class, Operation | Cannot be further specialized / overridden |

### Combining Tagged Values

Multiple tagged values are comma-separated inside one pair of braces:

```
employees : List<Employee> {ordered, unique}
```

This means the collection keeps insertion order and rejects duplicates -- behaves like a `LinkedHashSet`.

### Tagged Values on Associations

```
    Department                          Employee
  +------------+                    +-------------+
  |            |  1        0..*     |             |
  |            |--------------------| {ordered}   |
  |            |  dept     members  |             |
  +------------+                    +-------------+
```

The `{ordered}` on the `members` end means the department maintains its employees in a defined sequence.

## 🔹 Constraints

Constraints are boolean conditions enclosed in curly braces `{ }` that restrict the values a model element can take. They can appear:

- Beside an attribute or operation
- Near an association
- Inside a note symbol attached to an element
- As a standalone annotation between elements

### Constraint Syntax

```
{ natural language constraint }
{ OCL expression }
```

### Common Inline Constraints

```
+-------------------------------------------+
|              Employee                      |
+-------------------------------------------+
| - salary : Decimal  {salary > 0}          |
| - age : int  {age >= 18 and age <= 120}   |
| - email : String  {not null}              |
+-------------------------------------------+
```

### Constraints Between Elements

A constraint between two associations or elements is drawn as a dashed line connecting them, with the constraint label:

```
  Person
  +--------+
  |        |----- wife     {xor}
  |        |----- husband
  +--------+
```

The `{xor}` constraint means a person can have a `wife` or a `husband` association, but not both simultaneously.

### OCL Basics (Object Constraint Language)

OCL is the formal constraint language of UML. You write it inside `{ }` or in a note attached to the element.

| OCL Construct | Meaning | Example |
|---------------|---------|---------|
| `self` | The current object | `self.age >= 18` |
| `inv:` | Class invariant | `inv: self.balance >= 0` |
| `pre:` | Operation precondition | `pre: amount > 0` |
| `post:` | Operation postcondition | `post: self.balance = self.balance@pre - amount` |
| `@pre` | Value before operation | `self.balance@pre` |
| `->` | Collection operation | `self.employees->size() > 0` |
| `->forAll()` | Universal quantifier | `self.items->forAll(i \| i.price > 0)` |
| `->exists()` | Existential quantifier | `self.roles->exists(r \| r.name = 'admin')` |
| `->select()` | Filter collection | `self.orders->select(o \| o.total > 100)` |
| `->collect()` | Map/project collection | `self.employees->collect(e \| e.name)` |
| `->isEmpty()` | Collection has no elements | `self.errors->isEmpty()` |
| `->notEmpty()` | Collection has elements | `self.items->notEmpty()` |
| `implies` | Logical implication | `self.isManager implies self.salary > 50000` |
| `and` / `or` / `not` | Logical operators | `self.age >= 18 and self.age <= 65` |
| `if-then-else-endif` | Conditional | `if self.vip then 0.1 else 0.0 endif` |

**Full OCL constraint example attached as a note:**

```
+---------------------+
|    BankAccount      |
+---------------------+      +- - - - - - - - - - - - - - - - -+
| - balance : Decimal |------| context BankAccount              |
+---------------------+      | inv: self.balance >= 0           |
| + withdraw(amt)     |      | pre: amt > 0 and amt <= balance  |
+---------------------+      | post: balance = balance@pre - amt|
                              +- - - - - - - - - - - - - - - - -+
```

## 🔹 Notes (Comment Symbol)

The UML note is a rectangle with a folded top-right corner (dog-ear). It attaches to any element via a dashed line.

### Note Symbol ASCII Art

```
  +-------------------------------+
  | This class handles all        |\
  | payment processing logic.     | |
  | See requirements doc R-042.   |/
  +-------------------------------+
         |
         | (dashed line)
         |
  +-------------------+
  | PaymentProcessor  |
  +-------------------+
```

A more compact representation often used:

```
  .-------------------------------.
  | Constraint: all items in the  |
  | order must belong to the same |
  | warehouse region.             |
  '-------------------------------'
         :
         : (dashed)
         :
```

### Notes Containing OCL

```
  +----------------------------------+
  | {context Order                   |\
  |  inv: self.items                 | |
  |    ->forAll(i |                  | |
  |      i.warehouse = self.region)  | |
  | }                                |/
  +----------------------------------+
         :
    +----------+
    |  Order   |
    +----------+
```

### When to Use Notes

- Explain design rationale that the diagram itself cannot convey
- Attach OCL constraints formally
- Reference external documents, requirements, or JIRA tickets
- Clarify ambiguous [[Relationships]] or unusual patterns
- Mark areas as TODO or under review

## 🔹 Multiplicity

Multiplicity defines how many instances participate in a [[Relationships|relationship]] or how many values an attribute holds.

### Multiplicity Reference Table

| Notation | Meaning | Example Scenario |
|----------|---------|-----------------|
| `1` | Exactly one | An `Order` has exactly `1` customer |
| `0..1` | Zero or one (optional) | A `Person` has `0..1` spouse |
| `*` | Zero or more | A `Folder` contains `*` files |
| `0..*` | Zero or more (explicit form of `*`) | Same as `*`, sometimes preferred for clarity |
| `1..*` | One or more (at least one) | A `Team` has `1..*` members |
| `n..m` | Between n and m inclusive | A `Car` has `4..5` wheels (including spare) |
| `n` | Exactly n | A `Bicycle` has `2` wheels |
| `2..4` | Between 2 and 4 | A `Bridge` game has `2..4` players |

### Multiplicity on Associations

Multiplicity is written near the **target end** of an association -- it describes how many target objects one source object links to.

```
  +------------+  1         0..*  +----------+
  | Department |------------------| Employee |
  +------------+                  +----------+
       1 department          0..* employees

  Reading: "One Department has zero or more Employees."
           "Each Employee belongs to exactly one Department."
```

### Multiplicity on Attributes

```
+-------------------------------+
|          Student               |
+-------------------------------+
| - name : String [1]           |   <-- exactly one name
| - phoneNumbers : String [0..*]|   <-- zero or more phone numbers
| - address : Address [1..3]    |   <-- between 1 and 3 addresses
+-------------------------------+
```

### Multiplicity with Tagged Values

Multiplicity and tagged values work together to specify collection semantics:

```
  +----------+  1        * {ordered, unique}  +---------+
  | Playlist |--------------------------------| Song    |
  +----------+                                +---------+

  Songs are ordered (sequence matters) and unique (no duplicates).
```

| Combination | Collection Type (Java equivalent) |
|-------------|----------------------------------|
| `{unique}` | `Set` |
| `{ordered, unique}` | `LinkedHashSet` / `SortedSet` |
| `{ordered}` (nonunique) | `List` |
| `{nonunique, unordered}` | `Bag` / `Multiset` |

## 🔹 Namespaces and Qualified Names

UML uses `::` (double colon) as the namespace separator, mirroring C++ conventions.

### Namespace Syntax

```
Package::SubPackage::ClassName
```

**Examples:**

| Qualified Name | Meaning |
|---------------|---------|
| `java::util::List` | `List` inside `util` inside `java` |
| `Model::Domain::Customer` | `Customer` class in `Domain` sub-package |
| `UI::Controls::Button` | `Button` in the `Controls` namespace |

### Namespaces in Diagrams

When a diagram shows classes from multiple packages, you can either:

1. **Use qualified names directly:**

```
  +---------------------------+         +---------------------------+
  | Domain::Order             |         | Persistence::OrderRepo    |
  +---------------------------+         +---------------------------+
  | - items : Domain::LineItem|         | + save(o : Domain::Order) |
  +---------------------------+         +---------------------------+
```

2. **Use package frames** (preferred for larger diagrams):

```
  +--[ Domain ]----------------------------+
  |                                        |
  |  +----------+        +------------+    |
  |  |  Order   |------->| LineItem   |    |
  |  +----------+        +------------+    |
  |                                        |
  +----------------------------------------+

  +--[ Persistence ]-----------------------+
  |                                        |
  |  +---------------+                     |
  |  |  OrderRepo    |                     |
  |  +---------------+                     |
  |                                        |
  +----------------------------------------+
```

### Namespace Visibility

Packages themselves can contain visibility markers on their members:

- `+ PublicClass` -- visible outside the package
- `- InternalHelper` -- private to the package

This connects to the `~` (package) visibility on class members -- a `~` member is accessible to all classes within the same namespace.

## 🔹 Stereotype Placement and Associations (Combined Example)

Bringing it all together -- stereotypes, visibility, multiplicity, constraints, tagged values, and notes on a single diagram:

```
                           <<interface>>
                          +------------------+
                          |    Printable     |
                          +------------------+
                          | + print() : void |
                          +------------------+
                                  ^
                                  | implements
                                  | (dashed)
                                  |
  +--[ Inventory ]----------------------------------------------+
  |                                                              |
  |  +-----------------------------+    1     0..*               |
  |  |       <<entity>>            |-----------.                 |
  |  |        Product              |           |  {ordered}      |
  |  +-----------------------------+           |                 |
  |  | - name : String             |     +-----'--------+       |
  |  | - price : Decimal {> 0}     |     |  <<entity>>  |       |
  |  | - sku : String {readOnly}   |     |   Review     |       |
  |  | # category : String        |     +--------------+       |
  |  +-----------------------------+     | - rating:int |       |
  |  | + getPrice() : Decimal      |     |   {1..5}     |       |
  |  | + applyDiscount(pct) {query}|     | - text:String|       |
  |  +-----------------------------+     +--------------+       |
  |         |                                                    |
  |         | (dashed, note)                                     |
  |         |                                                    |
  |  +------------------------------+                           |
  |  | SKU is assigned at creation  |\                          |
  |  | and never changes. Format:   | |                          |
  |  | [A-Z]{3}-[0-9]{6}           |/                          |
  |  +------------------------------+                           |
  |                                                              |
  +--------------------------------------------------------------+
```

**Reading this diagram:**

- `Product` implements `Printable` (dashed arrow = realization, see [[Relationships]])
- `Product` is stereotyped `<<entity>>` (persistent domain object)
- `sku` is `{readOnly}` -- set once, never modified
- `price` has an inline constraint `{> 0}`
- `applyDiscount` is `{query}` -- it returns a value but does not mutate state
- `Product` has `0..*` `Review` objects in `{ordered}` sequence
- `Review.rating` has a constraint `{1..5}`
- A note explains the SKU format rule
- Everything lives inside the `Inventory` package frame

See also: [[Relationships]], [[Class Diagram]], [[Use Case Diagram]], [[Sequence Diagram]], [[Activity Diagram]], [[State Machine Diagram]]
