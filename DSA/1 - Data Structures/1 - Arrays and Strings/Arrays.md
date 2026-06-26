---
tags:
  - algorithms
  - data-structure
  - array
---

## 🔹 Real-World Analogy

Think of an array like a **row of numbered lockers** in a hallway. Each locker has a fixed position (index), is the same size, and sits right next to its neighbor. If you know the locker number, you can walk straight to it -- no searching required. But if you want to insert a new locker in the middle, you have to physically shift every locker after it one position to the right to make room.

This is exactly how arrays work in memory: contiguous, fixed-size slots, instant access by position, but expensive rearrangement.

---

## 🔹 Definition

An **array** is a collection of elements stored in **contiguous (adjacent) memory locations**, where each element is the same fixed size and can be accessed directly by its **index**.

Key properties:
- **Homogeneous** -- all elements are the same type (and therefore the same size in memory)
- **Contiguous** -- elements are stored back-to-back with no gaps
- **Indexed** -- each element is identified by a zero-based integer position
- **Fixed or dynamic size** -- depending on static vs dynamic arrays

Arrays are the most fundamental data structure. Nearly every other data structure (hash tables, heaps, stacks, queues) can be built on top of arrays.

---

## 🔹 Static vs Dynamic Arrays

| Property              | Static Array                          | Dynamic Array                                |
| --------------------- | ------------------------------------- | -------------------------------------------- |
| **Size**              | Fixed at compile time                 | Grows/shrinks at runtime                     |
| **Declaration**       | `int arr[10];`                        | `std::vector<int> v;`                        |
| **Memory**            | Stack (usually)                       | Heap (internal buffer)                       |
| **Resize**            | Not possible                          | Automatic (doubles capacity)                 |
| **Overhead**          | None                                  | Extra capacity + metadata (size, capacity)   |
| **Use when**          | Size is known and constant            | Size is unknown or changes over time         |
| **Examples**          | C arrays, C++ `std::array`            | C++ `std::vector`, C# `List<T>`, Java `ArrayList` |

**Rule of thumb**: If the size is known at compile time and will never change, use a static array. Otherwise, use a dynamic array.

---

## 🔹 Memory Layout

Arrays occupy a single, unbroken block of memory. Each element sits at a predictable offset from the base address.

```
  Index:       [0]        [1]        [2]        [3]        [4]
             +----------+----------+----------+----------+----------+
  Value:     |    10    |    20    |    30    |    40    |    50    |
             +----------+----------+----------+----------+----------+
  Address:   0x1000     0x1004     0x1008     0x100C     0x1010
                 |          |          |          |          |
                 +--- 4B ---+--- 4B ---+--- 4B ---+--- 4B ---+

  Base address = 0x1000
  Element size = 4 bytes (int)
  Total memory = 5 elements x 4 bytes = 20 bytes
```

The contiguous layout is why arrays are **cache-friendly**. When the CPU loads `arr[0]` into the cache, neighboring elements `arr[1]`, `arr[2]`, etc. come along for free (spatial locality). This makes sequential array traversal one of the fastest operations a computer can do.

---

## 🔹 Indexing -- O(1) Random Access

The address of any element can be computed directly with arithmetic:

```
address(arr[i]) = base_address + i * element_size
```

Example with `int arr[5]` starting at address `0x1000`:

```
arr[0] = 0x1000 + 0 * 4 = 0x1000
arr[1] = 0x1000 + 1 * 4 = 0x1004
arr[2] = 0x1000 + 2 * 4 = 0x1008
arr[3] = 0x1000 + 3 * 4 = 0x100C
arr[4] = 0x1000 + 4 * 4 = 0x1010
```

No matter how large the array is -- 10 elements or 10 million -- accessing any element by index is a single multiplication and addition. This is **O(1) constant time**.

This is the defining advantage of arrays over [[Singly Linked List|linked lists]], which require O(n) traversal to reach an arbitrary element.

---

## 🔹 Operations in Detail

### Access by Index -- O(1)

```cpp
int value = arr[3];  // Directly computed: base + 3 * sizeof(int)
```

This is the fastest possible data access. One arithmetic operation, one memory read.

---

### Search (Unsorted) -- O(n)

When the array is unsorted, you have no choice but to check every element:

```cpp
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target)
            return i;   // Found at index i
    }
    return -1;          // Not found
}
```

**Worst case**: target is the last element or not present -- you scan all n elements.

---

### Search (Sorted) -- O(log n)

If the array is sorted, **binary search** eliminates half the remaining elements with each comparison:

```cpp
int binarySearch(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoids overflow vs (left+right)/2
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

Each iteration cuts the search space in half: n -> n/2 -> n/4 -> ... -> 1. This takes log2(n) steps.

For n = 1,000,000 elements: linear search checks up to 1,000,000 elements; binary search checks at most 20.

---

### Insert at End (Dynamic Array) -- Amortized O(1)

```cpp
std::vector<int> v = {10, 20, 30};
v.push_back(40);  // Appends 40 at the end
```

If the internal buffer has spare capacity, this is a single write -- O(1). If the buffer is full, the dynamic array allocates a new buffer (typically 2x the old size), copies everything over, then appends. The copy is O(n), but it happens so rarely that the cost averages out to O(1) amortized. (See the amortized analysis section below.)

---

### Insert at Arbitrary Position -- O(n)

To insert at index `k`, every element from index `k` onward must shift one position to the right:

```
Before inserting 25 at index 2:

  [0]    [1]    [2]    [3]    [4]
+------+------+------+------+------+
|  10  |  20  |  30  |  40  |  50  |
+------+------+------+------+------+

Step 1: Shift elements right starting from the end

  [0]    [1]    [2]    [3]    [4]    [5]
+------+------+------+------+------+------+
|  10  |  20  |  30  |  30  |  40  |  50  |    arr[5] = arr[4]
+------+------+------+------+------+------+    arr[4] = arr[3]
                  ^                             arr[3] = arr[2]
                  |
Step 2: Write the new value

  [0]    [1]    [2]    [3]    [4]    [5]
+------+------+------+------+------+------+
|  10  |  20  |  25  |  30  |  40  |  50  |    arr[2] = 25
+------+------+------+------+------+------+
```

```cpp
// Insert value at index k in array of size n
void insertAt(int arr[], int& n, int k, int value) {
    for (int i = n; i > k; i--) {
        arr[i] = arr[i - 1];  // Shift right
    }
    arr[k] = value;
    n++;
}
```

**Worst case**: inserting at index 0 shifts all n elements -- O(n).

---

### Delete at Arbitrary Position -- O(n)

To delete at index `k`, every element after `k` must shift one position to the left:

```
Before deleting element at index 2 (value 30):

  [0]    [1]    [2]    [3]    [4]
+------+------+------+------+------+
|  10  |  20  |  30  |  40  |  50  |
+------+------+------+------+------+

Step 1: Shift elements left starting from index 2

  [0]    [1]    [2]    [3]    [4]
+------+------+------+------+------+
|  10  |  20  |  40  |  50  |  50  |    arr[2] = arr[3]
+------+------+------+------+------+    arr[3] = arr[4]

Step 2: Decrease size (logical delete -- ignore last slot)

  [0]    [1]    [2]    [3]
+------+------+------+------+
|  10  |  20  |  40  |  50  |
+------+------+------+------+
```

```cpp
// Delete element at index k in array of size n
void deleteAt(int arr[], int& n, int k) {
    for (int i = k; i < n - 1; i++) {
        arr[i] = arr[i + 1];  // Shift left
    }
    n--;
}
```

**Worst case**: deleting at index 0 shifts all n-1 elements -- O(n).

**Trick**: If order doesn't matter, you can swap the target element with the last element and shrink -- O(1). This is sometimes called "swap and pop."

---

## 🔹 Dynamic Array Resizing -- Amortized O(1)

Dynamic arrays (like `std::vector`) start with some initial capacity and **double** when full. This is the key insight behind amortized O(1) append.

### The Doubling Pattern

```
Operation    Size    Capacity    Cost (copies during resize)
---------    ----    --------    ---------------------------
push(A)       1        1         0  (no resize needed)
push(B)       2        2         1  (resize: copy 1 element)
push(C)       3        4         2  (resize: copy 2 elements)
push(D)       4        4         0
push(E)       5        8         4  (resize: copy 4 elements)
push(F)       6        8         0
push(G)       7        8         0
push(H)       8        8         0
push(I)       9       16         8  (resize: copy 8 elements)
...
```

### Why It's Amortized O(1)

After n insertions, the total number of copies from all resizes is:

```
1 + 2 + 4 + 8 + ... + n/2 + n  =  2n - 1
```

So the **average cost per insertion** = (2n - 1) / n, which is approximately 2, which is O(1).

Think of it this way: each element "pays" for its own insertion plus helps pay for a future resize. The expensive resizes happen exponentially less often, so the cost is spread out.

### The Tradeoff: Wasted Space

The downside of doubling is **up to 50% wasted capacity**. Right after a resize from capacity 8 to 16, you have 9 elements in a buffer that holds 16 -- nearly half the memory is unused.

Some implementations use a growth factor of 1.5x instead of 2x to reduce waste at the cost of slightly more frequent resizes. The amortized O(1) guarantee holds for any constant growth factor > 1.

---

## 🔹 2D Arrays / Matrices

A 2D array is logically a grid of rows and columns, but in memory it's still stored as a flat, contiguous block.

### Row-Major Order (C, C++, C#, Java)

Elements are stored **row by row**:

```
Logical view:           Memory layout (row-major):

     col 0  col 1  col 2
row 0 [ A ]  [ B ]  [ C ]     A  B  C  D  E  F  G  H  I
row 1 [ D ]  [ E ]  [ F ]     |-row 0-|--row 1-|--row 2-|
row 2 [ G ]  [ H ]  [ I ]

  Address of arr[i][j] = base + (i * num_cols + j) * element_size
```

### Column-Major Order (Fortran, MATLAB, R)

Elements are stored **column by column**:

```
Memory layout (column-major):

  A  D  G  B  E  H  C  F  I
  |-col 0-|--col 1-|--col 2-|

  Address of arr[i][j] = base + (j * num_rows + i) * element_size
```

### Why This Matters

Accessing elements in the "wrong" order causes **cache misses**. In a row-major language like C++, iterate over rows in the inner loop:

```cpp
// GOOD: row-major traversal (cache-friendly in C++)
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++)
        process(arr[i][j]);

// BAD: column-major traversal (cache-unfriendly in C++)
for (int j = 0; j < cols; j++)
    for (int i = 0; i < rows; i++)
        process(arr[i][j]);
```

For large matrices, the performance difference can be 10x or more due to cache behavior.

---

## 🔹 Time Complexity Summary

| Operation                   | Time Complexity | Notes                                         |
| --------------------------- | --------------- | --------------------------------------------- |
| **Access by index**         | O(1)            | Direct address calculation                    |
| **Search (unsorted)**       | O(n)            | Linear scan                                   |
| **Search (sorted)**         | O(log n)        | Binary search                                 |
| **Insert at beginning**     | O(n)            | Shift all elements right                      |
| **Insert at middle**        | O(n)            | Shift elements from insertion point            |
| **Insert at end (static)**  | O(1)            | If space is available                          |
| **Insert at end (dynamic)** | O(1) amortized  | Occasional O(n) resize                        |
| **Delete at beginning**     | O(n)            | Shift all elements left                       |
| **Delete at middle**        | O(n)            | Shift elements from deletion point             |
| **Delete at end**           | O(1)            | Just decrement size                            |
| **Append (dynamic)**        | O(1) amortized  | Same as insert at end                          |

---

## 🔹 Space Complexity

| Scenario            | Space       | Notes                                                    |
| ------------------- | ----------- | -------------------------------------------------------- |
| **Static array**    | O(n)        | Exactly n elements, no overhead                          |
| **Dynamic array**   | O(n)        | Up to 2n allocated due to doubling strategy              |
| **Auxiliary space** | O(1)        | Most array operations are in-place (no extra allocation) |

---

## 🔹 When to Use Arrays (and When Not To)

### Use arrays when:
- You need **fast random access** by index -- O(1)
- The data size is **known** or **roughly predictable**
- You're doing **sequential iteration** (cache-friendly)
- You need a **simple, low-overhead** container
- You're building other data structures on top (heaps, hash tables)

### Consider alternatives when:
- You need **frequent insertions/deletions at the front or middle** -- use a [[Singly Linked List]] instead
- You need **O(1) lookup by key** -- use a [[Hash Map and Hash Set]]
- You need **LIFO access** -- use a [[Stack]] (often built on an array anyway)
- You need **FIFO access** -- use a [[Queue]] or deque
- The data size is **highly unpredictable and sparse** -- consider a linked list or hash map
- You need **sorted data with frequent inserts** -- consider a balanced BST or skip list

---

## 🔹 Common Pitfalls

**1. Off-by-one errors**

The most common bug in array code. Arrays are 0-indexed, so valid indices for an array of size n are `0` to `n-1`.

```cpp
// BUG: accessing arr[n] is out of bounds
for (int i = 0; i <= n; i++)   // Should be i < n
    process(arr[i]);
```

**2. Out-of-bounds access**

C/C++ won't stop you from accessing memory outside the array. This causes undefined behavior -- crashes, corrupted data, or silent bugs.

```cpp
int arr[5];
arr[5] = 99;   // UB! Valid indices are 0..4
arr[-1] = 99;  // UB! Negative index
```

**3. Modifying an array while iterating**

Inserting or deleting elements during a forward traversal can cause skipped elements or infinite loops.

```cpp
// BUG: after erasing, the next element shifts into position i
//       but i increments, so that element is skipped
for (int i = 0; i < v.size(); i++) {
    if (shouldRemove(v[i])) {
        v.erase(v.begin() + i);
        // Fix: add i-- here, or iterate backward
    }
}
```

**4. Confusing size vs capacity (dynamic arrays)**

`size` is how many elements are stored. `capacity` is how much memory is allocated. Accessing indices between size and capacity is undefined behavior even though the memory exists.

**5. Forgetting that array assignment copies in some languages**

In C++, assigning a `std::array` copies all elements. In C, raw arrays can't be assigned at all. In languages like Python/Java, assigning an array variable copies the reference, not the data.

---

## 🔹 Template Code

### Basic Array Traversal

```cpp
void traverse(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        // Process arr[i]
    }
}
```

### Binary Search on Sorted Array

```cpp
// Returns index of target, or -1 if not found
int binarySearch(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return -1;
}
```

### In-Place Reversal (Two-Pointer Technique)

```cpp
void reverse(int arr[], int n) {
    int left = 0, right = n - 1;
    while (left < right) {
        // Swap arr[left] and arr[right]
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

```
Before:  [10] [20] [30] [40] [50]
          L                    R      swap(10, 50)

Step 1:  [50] [20] [30] [40] [10]
               L          R           swap(20, 40)

Step 2:  [50] [40] [30] [20] [10]
                    L=R               left >= right, stop

After:   [50] [40] [30] [20] [10]
```

---

## 🔹 Related Concepts

- [[Strings]] -- strings are arrays of characters with specialized operations
- [[Hash Map and Hash Set]] -- use arrays internally (bucket array); O(1) average lookup by key
- [[Singly Linked List]] -- the alternative when frequent insertion/deletion matters more than random access
- [[Stack]] -- LIFO structure often implemented with an array
- [[Queue]] -- FIFO structure; circular buffer variant uses an array
- [[Big O - Definition]] -- understanding time complexity is essential for comparing array operations
