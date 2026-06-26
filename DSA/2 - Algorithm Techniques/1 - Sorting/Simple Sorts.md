---
tags:
  - algorithms
  - sorting
  - fundamental
---

# Simple Sorts

Three foundational O(n^2) sorting algorithms. They are rarely used on large datasets in production, but understanding them is essential: they appear in interviews, they teach core sorting mechanics, and **Insertion Sort** is genuinely useful for small or nearly-sorted data (it is the subroutine inside Timsort and Introsort).

All walkthroughs use the same input array: **[5, 3, 8, 1, 2]**

---

## 🔹 Bubble Sort

**Real-world analogy:** Bubbles rising in water — the largest element "bubbles up" to the end of the array each pass.

**How it works:** Repeatedly walk through the array comparing adjacent pairs. If the left element is larger than the right, swap them. After each full pass, the next-largest element is in its final position. Repeat until no swaps occur.

### Step-by-Step Walkthrough

```
Input: [5, 3, 8, 1, 2]

--- Pass 1 ---
Compare 5 > 3? YES → swap    [3, 5, 8, 1, 2]
Compare 5 > 8? NO            [3, 5, 8, 1, 2]
Compare 8 > 1? YES → swap    [3, 5, 1, 8, 2]
Compare 8 > 2? YES → swap    [3, 5, 1, 2, 8]  ← 8 is now in final position
                                            ^

--- Pass 2 ---
Compare 3 > 5? NO            [3, 5, 1, 2, 8]
Compare 5 > 1? YES → swap    [3, 1, 5, 2, 8]
Compare 5 > 2? YES → swap    [3, 1, 2, 5, 8]  ← 5 is now in final position
                                         ^

--- Pass 3 ---
Compare 3 > 1? YES → swap    [1, 3, 2, 5, 8]
Compare 3 > 2? YES → swap    [1, 2, 3, 5, 8]  ← 3 is now in final position
                                      ^

--- Pass 4 ---
Compare 1 > 2? NO            [1, 2, 3, 5, 8]
No swaps occurred → STOP EARLY (optimization)

Result: [1, 2, 3, 5, 8]
```

### Key Optimization

Track whether any swap happened during a pass. If a full pass completes with zero swaps, the array is sorted — exit early. This gives O(n) best-case on already-sorted input.

### Template Code

```cpp
void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;  // early termination
    }
}
```

### Common Pitfalls

- Forgetting the `n - 1 - i` bound (inner loop shrinks each pass because the tail is already sorted)
- Not implementing the early-stop optimization — without it, best case is still O(n^2)

---

## 🔹 Selection Sort

**Real-world analogy:** Picking cards from a messy pile — each time you scan for the smallest card and place it next in line.

**How it works:** Divide the array into a sorted portion (left) and unsorted portion (right). Repeatedly find the minimum element in the unsorted portion and swap it with the first unsorted element.

### Step-by-Step Walkthrough

```
Input: [5, 3, 8, 1, 2]
       |--- unsorted ---|

--- Step 1: Find min in [5, 3, 8, 1, 2] → min is 1 (index 3) ---
Swap arr[0] with arr[3]
[1, 3, 8, 5, 2]
 ^  |-- unsorted --|
sorted

--- Step 2: Find min in [3, 8, 5, 2] → min is 2 (index 4) ---
Swap arr[1] with arr[4]
[1, 2, 8, 5, 3]
 ^  ^  |unsorted|
sorted

--- Step 3: Find min in [8, 5, 3] → min is 3 (index 4) ---
Swap arr[2] with arr[4]
[1, 2, 3, 5, 8]
 ^  ^  ^ |uns|
 sorted

--- Step 4: Find min in [5, 8] → min is 5 (index 3) ---
Already in place, no swap needed
[1, 2, 3, 5, 8]
 ^  ^  ^  ^ |u|
  sorted

Result: [1, 2, 3, 5, 8]
```

### Template Code

```cpp
void selectionSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx])
                minIdx = j;
        }
        if (minIdx != i)
            swap(arr[i], arr[minIdx]);
    }
}
```

### Key Properties

- Always performs exactly n(n-1)/2 comparisons regardless of input order
- Minimizes the number of **swaps** — exactly n-1 swaps in the worst case (useful when writes are expensive)
- **NOT stable** — the long-range swap can change relative order of equal elements

### Common Pitfalls

- Assuming it is stable (it is not — the swap skips over equal elements)
- No early termination possible — always O(n^2) even on sorted input

---

## 🔹 Insertion Sort

**Real-world analogy:** Sorting a hand of playing cards — you pick up each new card and slide it into the correct position among the cards you already hold.

**How it works:** Iterate from the second element. For each element, compare it backwards through the sorted portion and shift larger elements right until you find its correct position. Insert it there.

### Step-by-Step Walkthrough

```
Input: [5, 3, 8, 1, 2]

--- Step 1: Insert 3 into sorted [5] ---
key = 3
5 > 3 → shift 5 right     [_, 5, 8, 1, 2]
Insert 3 at position 0     [3, 5, 8, 1, 2]
                             ^  ^
                            sorted

--- Step 2: Insert 8 into sorted [3, 5] ---
key = 8
5 > 8? NO → stop
Insert 8 at position 2     [3, 5, 8, 1, 2]
                             ^  ^  ^
                             sorted

--- Step 3: Insert 1 into sorted [3, 5, 8] ---
key = 1
8 > 1 → shift right        [3, 5, _, 8, 2]
5 > 1 → shift right        [3, _, 5, 8, 2]
3 > 1 → shift right        [_, 3, 5, 8, 2]
Insert 1 at position 0     [1, 3, 5, 8, 2]
                             ^  ^  ^  ^
                              sorted

--- Step 4: Insert 2 into sorted [1, 3, 5, 8] ---
key = 2
8 > 2 → shift right        [1, 3, 5, _, 8]
5 > 2 → shift right        [1, 3, _, 5, 8]
3 > 2 → shift right        [1, _, 3, 5, 8]
1 > 2? NO → stop
Insert 2 at position 1     [1, 2, 3, 5, 8]
                             ^  ^  ^  ^  ^
                               sorted

Result: [1, 2, 3, 5, 8]
```

### Template Code

```cpp
void insertionSort(int arr[], int n) {
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];  // shift right
            j--;
        }
        arr[j + 1] = key;  // insert
    }
}
```

### Why Insertion Sort Matters

- **O(n) on nearly sorted data** — if every element is at most k positions from its final spot, runtime is O(nk)
- Used as the base case in hybrid sorts like **Timsort** and **Introsort** (for subarrays below ~16-32 elements)
- Very low overhead — no recursion, no extra memory, simple inner loop
- **Stable** — equal elements maintain their relative order
- **Online** — can sort elements as they arrive (no need to know the full array upfront)

### Common Pitfalls

- Off-by-one: the outer loop starts at `i = 1`, not `i = 0`
- Shifting vs. swapping: shifting (move elements right, insert key once) is faster than repeated adjacent swaps

---

## 🔹 Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable? | In-Place? | Best For |
|---|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Educational; detecting sorted input |
| Selection Sort | O(n^2) | O(n^2) | O(n^2) | O(1) | No | Yes | Minimizing swaps (costly writes) |
| Insertion Sort | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Small arrays; nearly sorted data |

> [!tip] In practice, **Insertion Sort** is the only simple sort that sees real use — as the small-array base case inside [[Merge Sort]] and [[Quick Sort]] hybrids.

---

**See also:** [[Merge Sort]] | [[Quick Sort]] | [[Sorting Comparison]]
