---
tags: algorithms, technique
---

### 🧠 Divide and Conquer – Overview

**Divide and Conquer** is a fundamental algorithmic strategy that works by:

1. **Dividing** the problem into smaller subproblems *of the same type*.
    
2. **Conquering** the subproblems by solving them recursively.
    
3. **Combining** their solutions to solve the original problem.
    

---

## 🔹 Real-Life Analogy

Imagine you want to sort a large stack of cards:

- Split it into two smaller stacks.
    
- Sort each stack separately.
    
- Merge the two sorted stacks together.
    

That's divide and conquer in action!

---

## 🔸 Structure of Divide and Conquer Algorithm

```text
function DnC(problem):
    if problem is small:
        solve directly
    else:
        divide problem into subproblems
        solve each subproblem recursively
        combine the results
```

---

## 🔹 Famous Examples

|Algorithm|Description|Time Complexity|
|---|---|---|
|**Merge Sort**|Divide array, sort halves, merge|O(n log n)|
|**Quick Sort**|Partition, sort subarrays|O(n log n) avg, O(n²) worst|
|**Binary Search**|Divide array in half each step|O(log n)|
|**Karatsuba**|Fast multiplication of large numbers|O(n^1.585)|
|**Strassen’s Algorithm**|Matrix multiplication|O(n^2.81)|
|**Fast Fourier Transform (FFT)**|Used in signal processing|O(n log n)|

---

## 🔹 Example: Merge Sort (Rust-like pseudocode)

```rust
fn merge_sort(arr: &mut [i32]) {
    if arr.len() <= 1 {
        return;
    }

    let mid = arr.len() / 2;
    let mut left = arr[..mid].to_vec();
    let mut right = arr[mid..].to_vec();

    merge_sort(&mut left);
    merge_sort(&mut right);

    merge(arr, &left, &right);
}
```

### Key Points:

- **Divide**: split array into two halves
    
- **Conquer**: recursively sort both halves
    
- **Combine**: merge sorted halves into a full sorted array
    

---

## 🔹 Why Use Divide and Conquer?

|Benefit|Description|
|---|---|
|**Efficiency**|Often improves time complexity|
|**Parallelizable**|Subproblems can be solved independently|
|**Simplicity for recursion**|Maps naturally to recursive structure|

---

## 🔸 General Time Complexity Formula

Divide and Conquer recurrence often follows this form:

```
T(n) = a * T(n/b) + f(n)
```

Where:

- `a` = number of subproblems
    
- `n/b` = size of each subproblem
    
- `f(n)` = time to divide and combine
    

Example (Merge Sort):

```
T(n) = 2 * T(n/2) + O(n) → O(n log n)
```

---

## 🔹 Summary

|Step|Action|
|---|---|
|**Divide**|Split the problem|
|**Conquer**|Solve each piece recursively|
|**Combine**|Merge or combine the sub-results|
