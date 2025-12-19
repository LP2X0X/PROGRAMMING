---
tags:
  - algorithms
  - term
  - fundamental
---

Here’s an **exhaustive overview of Big O notation**, including **what it is, why it matters, common classes**, and **how to analyze it**, with **practical examples**.

---

## 🔹 What Is Big O Notation?

**Big O notation** describes the **upper bound** of the growth rate of an algorithm’s **time** or **space complexity** as the input size grows. It answers the question:

> “How does the algorithm's performance scale with increasing input size (n)?”

It **abstracts away constants and low-order terms**, focusing on the most significant growth factor.

![[Pasted image 20250715083302.png|center|600]]

---

## 🔹 Why It Matters

Big O helps you:

- **Predict scalability** of algorithms.
    
- **Compare efficiencies** independent of hardware.
    
- **Optimize** code or choose the right algorithm/data structure.
    

---

## 🔹 The Formal Definition

If a function `f(n)` is **O(g(n))**, it means:

> There exist constants `c > 0` and `n₀ ≥ 0` such that for all `n ≥ n₀`,  
> **f(n) ≤ c × g(n)**

This definition ensures that `f(n)` doesn't grow faster than `g(n)` (up to a constant factor) as `n` becomes large.

---

## 🔹 Common Big O Complexities (Time or Space)

|**Notation**|**Name**|**Example**|
|---|---|---|
|**O(1)**|Constant time|Array access `arr[i]`|
|**O(log n)**|Logarithmic time|Binary search|
|**O(n)**|Linear time|Single loop over an array|
|**O(n log n)**|Log-linear time|Merge sort, QuickSort average|
|**O(n²)**|Quadratic time|Nested loops (Bubble sort)|
|**O(n³)**|Cubic time|3 nested loops (e.g., matrix mult.)|
|**O(2ⁿ)**|Exponential time|Naive recursive Fibonacci|
|**O(n!)**|Factorial time|Permutation generation|

---

## 🔹 How to Determine Big O

### 1. **Count Dominant Operations**

Focus on the most repeated or recursive step.

```cpp
for (int i = 0; i < n; i++)    // O(n)
    sum += arr[i];
```

### 2. **Drop Constants**

```cpp
int a = arr[0];    // O(1)
int b = arr[1];    // O(1)
```

→ Combined: **O(1)**

### 3. **Additive Rules**

Separate independent code blocks:

```cpp
doSomethingA(n);  // O(n)
doSomethingB(n);  // O(n²)
```

→ Total = **O(n²)** (keep the larger one)

### 4. **Multiplicative Rule for Nested Loops**

```cpp
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        ...      // O(n²)
```

### 5. **Recursive Functions**

Use the **Recurrence Relation**:

```cpp
T(n) = 2T(n/2) + O(n)   // Merge sort
→ T(n) = O(n log n)
```

Use the **Master Theorem**:

> T(n) = aT(n/b) + f(n)  
> → Based on how f(n) compares to `n^log_b(a)`

---

## 🔹 Space Complexity

Measures memory growth with input size.

Examples:

- Array of size `n`: O(n)
    
- Recursive depth: O(n) (for recursion stack)
    
- Constant memory: O(1)
    

---

## 🔹 Best, Worst, Average Case

- **Best case**: Rarely useful unless guaranteed.
    
- **Worst case**: Used in Big O — the max time required.
    
- **Average case**: Sometimes useful but requires probabilistic analysis.
    

Example:

```cpp
int linearSearch(int arr[], int x, int n) {
    for (int i = 0; i < n; i++)
        if (arr[i] == x)
            return i;
    return -1;
}
```

- Best: O(1) (found early)
    
- Worst: O(n) (found at end or not found)
    
- Average: O(n)
    

---

## 🔹 Visual Growth Comparison

Here's how various complexities grow:

```
n = 10     →  O(1):1   O(log n):3  O(n):10   O(n log n):30   O(n²):100   O(2ⁿ):1024   O(n!):3628800
n = 100    →  O(1):1   O(log n):7  O(n):100  O(n log n):700  O(n²):10⁴   O(2ⁿ):10³⁰   O(n!):10¹⁵⁷
```

---

## 🔹 Non-Standard / Advanced Big O Cases

- **Amortized Complexity**: Averaged over multiple operations (e.g., appending to a dynamic array is O(1) _amortized_).
    
- **Probabilistic Complexity**: Based on expected values (e.g., hash table lookups).
    
- **Sublinear**: Algorithms faster than O(n) (e.g., binary search is O(log n)).
    

---

## 🔹 Examples in Practice

### 1. Binary Search

```cpp
int binarySearch(int arr[], int x, int n) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (arr[mid] == x) return mid;
        else if (arr[mid] < x) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

➡️ **O(log n)** — halves the search space each time

---

### 2. Merge Sort

```cpp
void mergeSort(int arr[], int l, int r) {
    if (l < r) {
        int m = l + (r - l)/2;
        mergeSort(arr, l, m);
        mergeSort(arr, m+1, r);
        merge(arr, l, m, r);
    }
}
```

➡️ Recurrence: T(n) = 2T(n/2) + O(n)  
➡️ **O(n log n)**

---

### 3. Naive Recursive Fibonacci

```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

➡️ **O(2ⁿ)** — exponential due to repeated subproblems

---

## 🔹 Tips for Writing Efficient Code

- Avoid nested loops if possible.
    
- Use efficient data structures (e.g., HashMap vs Array).
    
- Watch for hidden costs: recursive calls, list insertions, etc.
    
- Use memoization or dynamic programming to reduce exponential problems.
    

---

## 🔹 Related Notations

|Notation|Meaning|
|---|---|
|**O(g(n))**|Upper bound (worst case)|
|**Ω(g(n))**|Lower bound (best case)|
|**Θ(g(n))**|Tight bound (both upper and lower)|
|**o(g(n))**|Strictly less than g(n)|
|**ω(g(n))**|Strictly greater than g(n)|