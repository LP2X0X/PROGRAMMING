---
tags: algorithms, data_structure
---

A **stack** is a fundamental data structure that follows the **LIFO (Last In, First Out)** principle. That means the **last element added** to the stack is the **first one removed**.

---

## 🔹 Stack Analogy

Think of a **stack of plates**:

- You **push** plates onto the stack.
    
- You **pop** plates from the top.
    
- You **cannot access** plates in the middle.
    
![[Pasted image 20250715143456.png|center|500]]

---

## 🔸 Core Operations

|Operation|Description|
|---|---|
|`push(x)`|Add `x` to the top|
|`pop()`|Remove and return the top item|
|`peek()` / `top()`|View the top item without removing|
|`is_empty()`|Check if the stack is empty|
|`size()`|Get number of elements|

---

## 🔹 Visual Example

```text
Stack (top at the right):
[1, 2, 3]   ← push(3)
   ↑ top
```

After `pop()`, stack becomes:

```text
[1, 2]
```

---

## 🔹 Stack in Code (C++, Rust, Python)

### ✅ C++ (STL)

```cpp
#include <stack>
std::stack<int> s;
s.push(10);
s.push(20);
s.pop();      // removes 20
int x = s.top(); // 10
```

---

### ✅ Rust

```rust
fn main() {
    let mut stack: Vec<i32> = Vec::new();

    stack.push(1);
    stack.push(2);
    println!("{:?}", stack.pop()); // Some(2)
    println!("{:?}", stack.last()); // Some(1) → like peek
}
```

---

### ✅ Python

```python
stack = []
stack.append(1)
stack.append(2)
stack.pop()    # 2
stack[-1]      # 1 → peek
```

---

## 🔹 When to Use a Stack?

|Use Case|Example|
|---|---|
|**Undo mechanisms**|Editors, Photoshop|
|**Function call stack**|Recursion|
|**Balanced symbols**|Matching `{}`, `[]`, `()`|
|**DFS**|Depth-first search in graphs/trees|
|**Expression parsing**|Infix → Postfix, Postfix evaluation|

---

## 🔹 Custom Stack (Rust Example)

```rust
struct Stack<T> {
    data: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        Stack { data: Vec::new() }
    }

    fn push(&mut self, item: T) {
        self.data.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.data.pop()
    }

    fn peek(&self) -> Option<&T> {
        self.data.last()
    }

    fn is_empty(&self) -> bool {
        self.data.is_empty()
    }
}
```

---

## 🔸 Summary

- **Stack = LIFO** structure
    
- Common operations: `push`, `pop`, `peek`
    
- Built-in support in most languages (or use vector/list)
    
- Used in many algorithms: parsing, backtracking, graph traversal
    
