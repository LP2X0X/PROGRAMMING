---
tags:
  - rust
  - datatype
  - enum
  - fundamental
---

In Rust, `enum` (short for **enumeration**) is a powerful type that lets you define a value that can be **one of several possible variants**, each possibly with associated data.

It’s **much more flexible than C-style enums**, because each variant can carry additional data — making them great for things like state machines, result types, and more.

---

## 🧱 Basic Syntax

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

```rust
let dir = Direction::Left; // The variants of the enum are namespaced under its identifier
```

You can use a `match` statement to act on it:

```rust
match dir {
    Direction::Up => println!("Going up!"),
    Direction::Down => println!("Going down!"),
    Direction::Left => println!("Turning left!"),
    Direction::Right => println!("Turning right!"),
}
```

---

## 📦 Enums with Data

Variants can also **store data**, like tuples or named fields.

```ad-note
There’s an advantage to using an enum rather than a struct: each variant can have different types and amounts of associated data.
```

### 🎒 Tuple-style:

```rust
enum Message {
    Quit,
    Move(i32, i32),
    Write(String),
}
```

```rust
let msg = Message::Move(10, 20);
```

### 🧾 Struct-style:

- Use curly bracket variants when:
	- You want **clarity** (e.g., `x: i32, y: i32` is more obvious than `(i32, i32)`)
	- You may extend fields in the future
	- You're working with **more than 1 or 2 fields**

```rust
enum Message {
    ChangeColor { r: u8, g: u8, b: u8 },
}
```

```rust
let msg = Message::ChangeColor { r: 255, g: 0, b: 0 };
```

---

## 🦾 Define methods on struct

- We’re also able to define methods on enums. Here’s a method named `call` that we could define on our `Message` enum:

```rust
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
```

- The body of the method would use `self` to get the value that we called the method on. In this example, we’ve created a variable `m` that has the value `Message::Write(String::from("hello"))`, and that is what `self` will be in the body of the `call` method when `m.call()` runs.

---

## 🧪 Pattern Matching with Data

```rust
let msg = Message::Move(10, 20);

match msg {
    Message::Quit => println!("Quit"),
    Message::Move(x, y) => println!("Move to ({}, {})", x, y),
    Message::Write(text) => println!("Text: {}", text),
    Message::ChangeColor { r, g, b } => println!("Color: {}, {}, {}", r, g, b),
}
```

---

## ✅ Enums in `Option` and `Result`

Rust’s standard library uses enums heavily:

### `Option<T>`:

```rust
let maybe = Some(5);     // enum Option: Some(T) or None

match maybe {
    Some(val) => println!("Got: {}", val),
    None => println!("No value"),
}
```

### `Result<T, E>`:

```rust
fn divide(x: i32, y: i32) -> Result<i32, String> {
    if y == 0 {
        Err("Divide by zero".into())
    } else {
        Ok(x / y)
    }
}
```

---

## 🧠 Enums are Sum Types

Rust enums are like **tagged unions** (sum types). Each variant is one of several options, optionally carrying data of different types.

This makes them more powerful than C enums or Java enums.

---

## 🛠️ Deriving Traits

To print enums with `{:?}`, use `#[derive(Debug)]`:

```rust
#[derive(Debug)]
enum Status {
    Ok,
    Error(String),
}
