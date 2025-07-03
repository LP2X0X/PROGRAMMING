---
tags: rust, control, flow, fundamental
---

The `match` statement in Rust is a **powerful control flow construct** that lets you compare a value against multiple patterns and execute code based on which pattern matches — similar to `switch` in other languages, but **much safer and more expressive**.

---

## 🧱 Basic Syntax

- Inside match are the `match` arms. An arm has two parts: a pattern and some code.

```rust
match value {
    pattern1 => expr1, // This is an arm
    pattern2 => expr2,
    _ => fallback_expr,
}
```

- The **first matching arm** is executed.
- Patterns must be **exhaustive** — you must cover all possible cases. ^4a7317
- You can use `_` (wildcard) as a catch-all.
</br>
- The code associated with each arm is an expression, and the resultant value of the expression in the matching arm is the value that gets returned for the entire `match` expression.

---

### ✅ Example: Matching integers

```rust
let number = 3;

match number {
    1 => println!("One"),
    2 => println!("Two"),
    3 => println!("Three"),
    _ => println!("Something else"),
}
```

---

## 🧠 Pattern Matching Works with Enums

### Example: `Option`

```rust
let maybe_name = Some("Alice");

match maybe_name {
    Some(name) => println!("Hello, {}!", name),
    None => println!("No name provided."),
}
```

---

## 🔄 `match` as an Expression

You can return a value from `match`:

```rust
let num = 2;
let message = match num {
    1 => "one",
    2 => "two",
    _ => "many",
};

println!("Message: {}", message);
```

> `match` is not just a control statement — it's also an **expression**.

---

## 🎯 Matching Multiple Patterns

```rust
match x {
    1 | 2 => println!("One or two"),
    3..=5 => println!("Three to five"),
    _ => println!("Other"),
}
```

- `|` is **OR**
    
- `3..=5` is a **range pattern**
    

---

## 🧩 Destructuring with `match`

### Struct:

```rust
struct Point { x: i32, y: i32 }

let p = Point { x: 10, y: 20 };

match p {
    Point { x, y: 20 } => println!("x={}, y=20", x),
    Point { x, y } => println!("Point at {},{}", x, y),
}
```

### Enum with data:

Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is how we can extract values out of enum variants.

```rust
enum Message {
    Quit,
    Move(i32, i32),
}

let m = Message::Move(3, 5);

match m {
    Message::Quit => println!("Quit"),
    Message::Move(x, y) => println!("Move to ({}, {})", x, y),
}
```

---

## 🔐 Match Guards

Use `if` in a match arm to add conditions:

```rust
let age = Some(21);

match age {
    Some(n) if n >= 18 => println!("Adult"),
    Some(n) => println!("Minor"),
    None => println!("No age"),
}
```

---

## 🚫 No Fallthrough (Unlike `switch` in C)

Each arm ends with `,` or `{}`, and **does not fall through**.

You must handle each case explicitly, which avoids bugs.

---

## 🔁 `_` Wildcard

Used to match "everything else" — like `default:` in other languages:

```rust
match val {
    0 => println!("Zero"),
    _ => println!("Something else"),
}
```

---

## 🧠 Summary

|Feature|Example|
|---|---|
|Simple match|`match x { 1 => ..., _ => ... }`|
|Enum matching|`Some(x)`, `None`|
|Multiple patterns|`1|
|Ranges|`1..=5 => ...`|
|Destructuring|`Point { x, y } => ...`|
|Guards (conditions)|`Some(x) if x > 10 => ...`|
|Expression result|`let y = match x { ... }`|
