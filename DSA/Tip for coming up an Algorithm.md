---
tags:
  - algorithms
  - tip
---

One of the most **common and powerful ways** people come up with algorithms is:

> **Trying small examples** → **Observing patterns** → **Generalizing a process** → **Formulating an algorithm**

But that's just one approach. Here is a more complete breakdown of **how people (especially experienced programmers and algorithm designers)** typically invent or derive algorithms.

---

## 🔹 1. **Test with Small Inputs (Pattern Discovery)**

This is where **most people start**, especially in competitive programming, problem solving, and algorithm design:

- **Try small inputs** (e.g., n = 1, 2, 3)
    
- **Manually compute results**
    
- **Look for repeated steps or structure**
    
- Ask: “What’s changing as input grows?”
    

### 🧠 Example: Fibonacci Numbers

Try:

```
fib(1) = 1
fib(2) = 1
fib(3) = 2
fib(4) = 3
fib(5) = 5
```

Pattern: `fib(n) = fib(n-1) + fib(n-2)`  
➡️ Leads to the **recursive definition**

---

## 🔹 2. **Mathematical Insight or Problem Constraints**

Sometimes the constraints or problem form **suggest a known method**.

- Does it ask for **shortest path**? → Graph + BFS/Dijkstra
    
- Does it involve **combinations**? → Use DP or Pascal's triangle
    
- Constraints like `n ≤ 20` → brute force is okay
    
- Constraints like `n ≤ 10⁵` → must use O(n log n) or better
    

### 🧠 Example: Sorted array + search → use **binary search** (O(log n))

---

## 🔹 3. **Use Known Techniques**

Experienced coders mentally match problems with **toolkits**:

- Sliding window
    
- Two pointers
    
- Divide and conquer
    
- Prefix sums
    
- Dynamic programming
    
- Greedy algorithms
    
- Union-Find / Disjoint Set
    

They think:

> “Have I seen a similar structure before?”

---

## 🔹 4. **Work Backward (Goal-Driven Thinking)**

For optimization problems:

- Start from the **desired final state**
    
- Work backward to understand **dependencies**
    

Example: You want to find the **minimum number of coins** to reach a total.

- What combinations of smaller totals could lead to this total?
    
- This backward process helps build up a **DP recurrence**.
    

---

## 🔹 5. **Use Recursion and Memoization First (Then Optimize)**

For many problems, it’s easy to describe the logic recursively.

- Recursion is often a direct mirror of your “manual thinking”
    
- Then you optimize it using **memoization** or **bottom-up DP**
    

---

## 🔹 6. **Greedy Choice Testing**

Try making **greedy choices** on small inputs and ask:

- “Does this always lead to an optimal result?”
    
- “Can I prove this choice never causes harm?”
    

If you can prove it, that’s a **greedy algorithm**.

---

## 🔹 7. **Transform the Problem**

Sometimes the key is to **reframe** the problem:

- Turn strings into graphs
    
- Model states using arrays or bitmasks
    
- Use modulo arithmetic
    
- Use binary search on the answer (parametric search)
    

---

## 🔹 Summary: How People Invent Algorithms

|Step|Method|
|---|---|
|1|Try small examples, spot patterns|
|2|Use constraints to guess time complexity|
|3|Map to known techniques|
|4|Build recursively, optimize via DP|
|5|Test greedy choices and prove correctness|
|6|Reframe or reduce to other problems|
|7|Prototype + test + refine|