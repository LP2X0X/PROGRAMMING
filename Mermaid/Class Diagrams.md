---
tags:
  - mermaid
  - class-diagram
  - diagram
---

# Class Diagrams

Mermaid class diagrams let you model the structure of a system by showing classes, their attributes, methods, and the relationships between them. They follow UML conventions and render directly in Obsidian's reading view, making them ideal for documenting object-oriented designs, design patterns, and domain models.

See also: [[Syntax Basics]] | [[Styling and Themes]]

---

## 🔹 Quick Reference: Relationships

| Relationship         | Syntax       | Description                                      |
| -------------------- | ------------ | ------------------------------------------------ |
| Inheritance          | `A --|> B`   | A inherits from B (A is a subclass of B)         |
| Inheritance (reverse)| `B <|-- A`   | B is inherited by A (same meaning, reversed)     |
| Composition          | `A *-- B`    | A is composed of B (strong lifecycle dependency) |
| Composition (reverse)| `B --* A`    | B composes A (same meaning, reversed)            |
| Aggregation          | `A o-- B`    | A has-a B (weak lifecycle, B can exist alone)    |
| Aggregation (reverse)| `B --o A`    | B aggregates into A (same meaning, reversed)     |
| Association          | `A --> B`    | A uses / is associated with B                    |
| Association (reverse)| `B <-- A`    | B is used by A (same meaning, reversed)          |
| Dependency           | `A ..> B`    | A depends on B (uses B temporarily)              |
| Dependency (reverse) | `B <.. A`    | B is depended upon by A (same meaning, reversed) |
| Realization          | `A ..\|> B`  | A implements / realizes B (interface)            |
| Realization (reverse)| `B <\|.. A`  | B is implemented by A (same meaning, reversed)   |
| Link (solid)         | `A -- B`     | Undirected solid link                            |
| Link (dashed)        | `A .. B`     | Undirected dashed link                           |

> **Tip:** Solid lines (`--`) represent structural relationships. Dashed lines (`..`) represent behavioral/temporary ones.

---

## 🔹 Quick Reference: Visibility Markers

| Symbol | Meaning            |
| ------ | ------------------ |
| `+`    | Public             |
| `-`    | Private            |
| `#`    | Protected          |
| `~`    | Package / Internal |

---

## 🔹 Defining Classes

There are two main ways to define a class:

**Inline syntax** -- just reference the class name in a relationship:

```
class ClassName
```

**Block syntax** -- define attributes and methods inside curly braces:

```
class ClassName {
    +String name
    -int age
    +getName() String
    -calculateAge() int
}
```

- **Properties** are written as `visibility Type name` (e.g., `+String name`)
- **Methods** are written as `visibility methodName() ReturnType` (e.g., `+getName() String`)
- If there is no return type, just write `+doSomething()`
- Parameters can be included: `+setName(String name) void`

```mermaid
classDiagram
    class Person {
        +String firstName
        +String lastName
        -int age
        -String email
        +getFullName() String
        +getAge() int
        -validateEmail(String email) bool
        +toString() String
    }
```

---

## 🔹 Visibility Markers in Detail

- `+` **Public** -- accessible from anywhere
- `-` **Private** -- accessible only within the class
- `#` **Protected** -- accessible within the class and its subclasses
- `~` **Package / Internal** -- accessible within the same package or assembly

Apply visibility markers to both properties and methods:

```mermaid
classDiagram
    class BankAccount {
        +String accountNumber
        -double balance
        #String ownerName
        ~String branchCode
        +deposit(double amount) void
        +getBalance() double
        -validateTransaction(double amount) bool
        #notifyOwner(String message) void
        ~getInternalId() String
    }
```

---

## 🔹 Relationships in Detail

### Inheritance: `A --|> B`

A child class inherits from a parent class. Represents an "is-a" relationship.

```mermaid
classDiagram
    class Animal {
        +String name
        +makeSound() void
    }
    class Dog {
        +String breed
        +fetch() void
    }
    class Cat {
        +bool isIndoor
        +purr() void
    }
    Dog --|> Animal
    Cat --|> Animal
```

### Composition: `A *-- B`

A strong "whole-part" relationship where the part cannot exist without the whole. When the whole is destroyed, the parts are destroyed too.

```mermaid
classDiagram
    class House {
        +String address
    }
    class Room {
        +String name
        +double area
    }
    House *-- Room : contains
```

### Aggregation: `A o-- B`

A weak "whole-part" relationship where the part can exist independently of the whole. Destroying the whole does not destroy the parts.

```mermaid
classDiagram
    class Team {
        +String teamName
    }
    class Player {
        +String name
        +int jerseyNumber
    }
    Team o-- Player : has
```

### Association: `A --> B`

A general "uses" relationship. One class knows about and can reference another.

```mermaid
classDiagram
    class Teacher {
        +String name
    }
    class Course {
        +String title
        +int credits
    }
    Teacher --> Course : teaches
```

### Dependency: `A ..> B`

A weaker relationship where one class temporarily uses another (e.g., as a method parameter or local variable), but does not hold a long-lived reference.

```mermaid
classDiagram
    class OrderService {
        +processOrder(Order order) void
    }
    class EmailService {
        +sendConfirmation(String to) void
    }
    OrderService ..> EmailService : uses
```

### Realization / Interface: `A ..|> B`

A class implements an interface or realizes an abstract contract.

```mermaid
classDiagram
    class IShape {
        <<interface>>
        +area() double
        +perimeter() double
    }
    class Circle {
        +double radius
        +area() double
        +perimeter() double
    }
    class Rectangle {
        +double width
        +double height
        +area() double
        +perimeter() double
    }
    Circle ..|> IShape
    Rectangle ..|> IShape
```

### All Relationships Together

```mermaid
classDiagram
    class Vehicle {
        +String make
        +start() void
    }
    class Car {
        +int doors
    }
    class Engine {
        +int horsepower
    }
    class Wheel {
        +int diameter
    }
    class Driver {
        +String license
    }
    class FuelStation {
        +refuel() void
    }
    class IDriveable {
        <<interface>>
        +drive() void
        +brake() void
    }

    Car --|> Vehicle : inherits
    Car *-- Engine : composition
    Car o-- Wheel : aggregation
    Car --> Driver : association
    Car ..> FuelStation : dependency
    Car ..|> IDriveable : realizes
```

---

## 🔹 Cardinality / Multiplicity

You can annotate relationships with cardinality labels to show how many instances participate:

```
A "1" --> "*" B : label
```

Common cardinality notations:

| Notation | Meaning              |
| -------- | -------------------- |
| `"1"`    | Exactly one          |
| `"0..1"` | Zero or one          |
| `"*"`    | Zero or more         |
| `"1..*"` | One or more          |
| `"0..*"` | Zero or more (explicit) |
| `"n"`    | A specific number    |

```mermaid
classDiagram
    class Department {
        +String name
    }
    class Employee {
        +String name
        +String role
    }
    class Project {
        +String title
        +Date deadline
    }
    Department "1" --> "1..*" Employee : employs
    Employee "0..*" --> "0..*" Project : works on
    Department "1" --> "0..*" Project : sponsors
```

---

## 🔹 Annotations

Annotations mark a class with a stereotype or classifier:

| Annotation          | Usage                               |
| ------------------- | ----------------------------------- |
| `<<interface>>`     | Marks a class as an interface       |
| `<<abstract>>`      | Marks a class as abstract           |
| `<<enumeration>>`   | Marks a class as an enum            |
| `<<service>>`       | Marks a class as a service          |

You place the annotation inside the class block:

```
class ClassName {
    <<interface>>
    +methodName() ReturnType
}
```

```mermaid
classDiagram
    class IRepository {
        <<interface>>
        +getById(int id) Entity
        +save(Entity entity) void
        +delete(int id) void
    }
    class Color {
        <<enumeration>>
        RED
        GREEN
        BLUE
    }
    class Shape {
        <<abstract>>
        +area() double*
        +perimeter() double*
    }
    class NotificationService {
        <<service>>
        +send(String message) void
        +schedule(String message, Date time) void
    }
```

---

## 🔹 Generic Types

Use `~T~` to denote generic type parameters (tildes replace angle brackets since `<>` conflicts with Mermaid's syntax):

```
class List~T~ {
    +add(T item) void
    +get(int index) T
}
```

Generics also work in relationships:

```mermaid
classDiagram
    class List~T~ {
        -T[] items
        +add(T item) void
        +get(int index) T
        +size() int
    }
    class Map~K, V~ {
        +put(K key, V value) void
        +get(K key) V
        +containsKey(K key) bool
    }
    class UserService {
        +getUsers() List~User~
        +getUserMap() Map~String, User~
    }
    class User {
        +String name
        +String email
    }
    UserService --> List~User~
    UserService --> Map~String, User~
    List~User~ --> User
```

---

## 🔹 Namespace Grouping

Use `namespace` blocks to visually group related classes:

```
namespace MyNamespace {
    class ClassA { }
    class ClassB { }
}
```

```mermaid
classDiagram
    namespace DataAccess {
        class IRepository {
            <<interface>>
            +getById(int id) Entity
            +save(Entity entity) void
        }
        class SqlRepository {
            -String connectionString
            +getById(int id) Entity
            +save(Entity entity) void
        }
    }
    namespace Domain {
        class Entity {
            +int id
            +DateTime createdAt
        }
        class User {
            +String name
            +String email
        }
    }
    SqlRepository ..|> IRepository
    User --|> Entity
    IRepository --> Entity
```

---

## 🔹 Real-World Examples

### OOP Hierarchy: Animals with Interface

A classic inheritance hierarchy with an interface contract. `Dog` and `Cat` inherit from `Animal` and implement `ISoundMaker`.

```mermaid
classDiagram
    class Animal {
        <<abstract>>
        +String name
        +int age
        +eat() void
        +sleep() void
    }
    class ISoundMaker {
        <<interface>>
        +makeSound() String
    }
    class Dog {
        +String breed
        +makeSound() String
        +fetch() void
    }
    class Cat {
        +bool isIndoor
        +makeSound() String
        +purr() void
    }
    class Parrot {
        +String[] vocabulary
        +makeSound() String
        +mimic(String phrase) void
    }

    Dog --|> Animal
    Cat --|> Animal
    Parrot --|> Animal
    Dog ..|> ISoundMaker
    Cat ..|> ISoundMaker
    Parrot ..|> ISoundMaker
```

### Design Pattern: Observer Pattern

The Observer pattern lets a subject notify multiple observers when its state changes. Observers subscribe to the subject and react to updates without tight coupling.

```mermaid
classDiagram
    class IObserver {
        <<interface>>
        +update(String event, Object data) void
    }
    class ISubject {
        <<interface>>
        +attach(IObserver observer) void
        +detach(IObserver observer) void
        +notify(String event) void
    }
    class EventBus {
        -List~IObserver~ observers
        -String state
        +attach(IObserver observer) void
        +detach(IObserver observer) void
        +notify(String event) void
        +setState(String state) void
    }
    class Logger {
        -String logFile
        +update(String event, Object data) void
    }
    class EmailNotifier {
        -String smtpServer
        +update(String event, Object data) void
    }
    class Dashboard {
        -Map~String, Object~ metrics
        +update(String event, Object data) void
        +render() void
    }

    EventBus ..|> ISubject
    Logger ..|> IObserver
    EmailNotifier ..|> IObserver
    Dashboard ..|> IObserver
    EventBus "1" --> "*" IObserver : notifies
```

### Domain Model: E-Commerce Order System

A realistic domain model showing how `Customer` places `Order`s, each containing `OrderItem`s that reference `Product`s. `Payment` is processed per order.

```mermaid
classDiagram
    class Customer {
        +int customerId
        +String name
        +String email
        +List~Order~ orderHistory
        +placeOrder(Order order) void
    }
    class Order {
        +int orderId
        +DateTime orderDate
        +OrderStatus status
        +getTotal() double
        +addItem(OrderItem item) void
        +removeItem(int itemId) void
    }
    class OrderItem {
        +int lineNumber
        +int quantity
        +double unitPrice
        +getSubtotal() double
    }
    class Product {
        +int productId
        +String name
        +String description
        +double price
        +int stockQuantity
        +isInStock() bool
    }
    class Payment {
        +int paymentId
        +double amount
        +String method
        +PaymentStatus status
        +process() bool
        +refund() bool
    }
    class OrderStatus {
        <<enumeration>>
        PENDING
        CONFIRMED
        SHIPPED
        DELIVERED
        CANCELLED
    }
    class PaymentStatus {
        <<enumeration>>
        PENDING
        COMPLETED
        FAILED
        REFUNDED
    }

    Customer "1" --> "0..*" Order : places
    Order "1" *-- "1..*" OrderItem : contains
    OrderItem "0..*" --> "1" Product : references
    Order "1" --> "0..1" Payment : paid via
    Order --> OrderStatus
    Payment --> PaymentStatus
```

---

**See also:** [[Syntax Basics]] | [[Flowcharts]] | [[Sequence Diagrams]] | [[Styling and Themes]]
