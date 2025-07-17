---
tags: algorithms, term, fundamental
---

Here's a clear and comprehensive overview of **recursive functions**, how they work, and when and how to use them effectively.

---

## 🔹 What Is a Recursive Function?

A **recursive function** is a function that **calls itself** in order to solve a problem by breaking it down into **smaller subproblems** of the same type.

---

## 🔹 Core Components of Recursion

1. **Base Case** – The condition under which the function **stops calling itself**.
    
2. **Recursive Case** – The part of the function where it **calls itself with a smaller/simpler input**.
    
![[Pasted image 20250715141549.png|center|400]]

---

## 🔹 Anatomy of a Recursive Function

```rust
fn factorial(n: u32) -> u32 {
    if n == 0 {
        1 // Base case
    } else {
        n * factorial(n - 1) // Recursive case
    }
}
```

---

## 🔹 How It Works (Call Stack)

Calling `factorial(3)`:

```
factorial(3)
→ 3 * factorial(2)
      → 2 * factorial(1)
            → 1 * factorial(0)
                  → 1 (base case)
```

The return values resolve **from the inside out**:

```
factorial(0) = 1
factorial(1) = 1 * 1 = 1
factorial(2) = 2 * 1 = 2
factorial(3) = 3 * 2 = 6
```

---

## 🔹 Key Properties of Recursive Functions

|Concept|Explanation|
|---|---|
|**Call stack**|Each call adds a new frame on the stack|
|**Base case**|Prevents infinite recursion|
|**Stack overflow**|Too many recursive calls → crash|
|**Tail recursion**|Optimizable in some languages (not Rust yet)|

---

## 🔹 Common Use Cases for Recursion

|Problem Type|Examples|
|---|---|
|Divide & conquer|Merge sort, quicksort|
|Tree/graph traversal|DFS, parsing expressions|
|Backtracking|N-Queens, Sudoku, permutations|
|Mathematical recurrence|Fibonacci, factorial|

---

## 🔹 Recursive vs Iterative

|Aspect|Recursion|Iteration|
|---|---|---|
|Simplicity|Often simpler, elegant|More control/faster|
|Overhead|Higher (stack usage)|Lower (no call stack)|
|Tail call opt|Possible in some languages|Always efficient|
|Rust status|No tail-call optimization|Prefer loops when deep recursion is likely|

---

## 🔹 Example: Recursive Fibonacci in Rust

```rust
fn fib(n: u32) -> u32 {
    if n <= 1 {
        n
    } else {
        fib(n - 1) + fib(n - 2)
    }
}
```

⚠️ This is elegant, but very inefficient: **O(2ⁿ)**. Better to use **memoization** or iteration for large `n`.

---

## 🔹 Recursive Tree Traversal (Rust Example)

```rust
struct Node {
    value: i32,
    left: Option<Box<Node>>,
    right: Option<Box<Node>>,
}

fn inorder(node: &Option<Box<Node>>) {
    if let Some(n) = node {
        inorder(&n.left);
        println!("{}", n.value);
        inorder(&n.right);
    }
}
```

---

## ✅ Tips for Writing Recursive Functions

- Always define a **clear base case**.
    
- Make sure the **input gets smaller** with each call.
    
- Consider **memoization** if you see repeated calls with the same input.
    
- For **deep recursion**, prefer **iteration** in Rust to avoid stack overflow.
    
