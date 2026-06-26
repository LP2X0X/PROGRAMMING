---
tags:
  - algorithms
  - data-structure
  - heap
---

## 🔹 Real-World Analogy

Think of a hospital emergency room. Patients are not served in the order they arrive — the most critical patient is always treated first. When a new patient walks in, the triage nurse evaluates their urgency and slots them into the right position. When the doctor finishes with one patient, the next most critical one is immediately available at the front.

A heap works the same way: the most important element (smallest in a min-heap, largest in a max-heap) is always sitting at the top, instantly accessible. When you add or remove elements, the structure reorganizes itself with minimal work to maintain this guarantee.

## 🔹 What Is a Heap?

A heap is a **complete binary tree** that satisfies the **heap property**:

- **Min-Heap**: Every parent node is **less than or equal to** its children. The minimum element is always at the root.
- **Max-Heap**: Every parent node is **greater than or equal to** its children. The maximum element is always at the root.

Two critical properties define a heap:

1. **Heap Property** — the ordering constraint between parent and children (described above).
2. **Shape Property** — the tree must be a **complete binary tree**: every level is fully filled except possibly the last, which is filled from left to right with no gaps.

A heap is **NOT** a [[Binary Search Tree]]. In a BST, left child < parent < right child. In a heap, there is no ordering relationship between siblings or between left and right children — only between parent and child.

```
         Min-Heap                    Max-Heap
            1                          9
          /   \                      /   \
         3     5                    7     8
        / \   /                   / \   /
       7   4 8                   3   5 1

  Parent <= Children          Parent >= Children
  Root = minimum              Root = maximum
```

## 🔹 When to Use a Heap

- You need fast access to the minimum or maximum element
- You need to repeatedly extract the min/max while inserting new elements
- Implementing a [[Priority Queue]]
- Finding the K-th largest or smallest element in a stream
- Sorting data (heapsort)
- Graph algorithms like Dijkstra's shortest path or Prim's MST
- Merging K sorted lists or streams
- Median maintenance in a data stream (two-heap technique)

## 🔹 Array Representation

The key insight that makes heaps efficient is that a **complete binary tree can be stored in a flat array** with no pointers. The shape property guarantees there are no gaps, so the array is compact.

For a node at index `i` (0-indexed):

| Relationship | Formula        |
|-------------|----------------|
| Left child  | `2 * i + 1`    |
| Right child | `2 * i + 2`    |
| Parent      | `(i - 1) / 2`  |

The root is always at index `0`.

Here is the same min-heap shown as both a tree and an array:

```
  TREE VIEW:                          ARRAY VIEW:
                                      
       1          depth 0             index:  0   1   2   3   4   5
     /   \                            value: [1] [3] [5] [7] [4] [8]
    3     5       depth 1                     |   |   |   |   |   |
   / \   /                                   |   |   |   |   |   |
  7   4 8         depth 2                    root |   |   |   |   |
                                                  |   |   |   |   |
  Mapping:                                        |   |   |   |   |
  Node 1 (i=0) -> children at 1,2                 |   |   |   |   |
  Node 3 (i=1) -> children at 3,4       left of 0 |   |   |   |   |
  Node 5 (i=2) -> children at 5         right of 0    |   |   |   |
  Node 7 (i=3) -> parent at 1                         left of 1   |
  Node 4 (i=4) -> parent at 1                         right of 1  |
  Node 8 (i=5) -> parent at 2                                left of 2
```

Why this works: a complete binary tree of `n` nodes uses exactly indices `0` through `n-1`. No memory is wasted on null pointers or empty slots.

## 🔹 Operations — Step by Step

### Insert (Sift Up / Bubble Up)

**Strategy**: Add the new element at the end of the array (maintaining the complete tree shape), then "bubble it up" by swapping with its parent until the heap property is restored.

**Example**: Insert `2` into this min-heap:

```
  STEP 0 — Starting heap:

       1
     /   \
    3     5
   / \   /
  7   4 8

  Array: [1, 3, 5, 7, 4, 8]
```

```
  STEP 1 — Add 2 at the end (index 6 = right child of node at index 2):

       1
     /   \
    3     5
   / \   / \
  7   4 8   2    <-- new element added here

  Array: [1, 3, 5, 7, 4, 8, 2]
                            ^
  Compare 2 with parent 5 (index (6-1)/2 = 2).  2 < 5, so swap.
```

```
  STEP 2 — Swap 2 and 5:

       1
     /   \
    3     2      <-- 2 moved up
   / \   / \
  7   4 8   5

  Array: [1, 3, 2, 7, 4, 8, 5]

  Compare 2 with parent 1 (index (2-1)/2 = 0).  2 > 1, STOP.
```

```
  FINAL — Heap property restored:

       1
     /   \
    3     2
   / \   / \
  7   4 8   5

  Array: [1, 3, 2, 7, 4, 8, 5]
```

**Time**: O(log n) — at most we travel from a leaf to the root, which is the height of the tree.

### Extract-Min (Sift Down)

**Strategy**: The min is at the root. Replace the root with the last element in the array (maintaining completeness), then "sift it down" by swapping with the smaller child until the heap property is restored.

**Example**: Extract-min from this min-heap:

```
  STEP 0 — Starting heap (extract root = 1):

       1         <- this is removed
     /   \
    3     2
   / \   / \
  7   4 8   5

  Array: [1, 3, 2, 7, 4, 8, 5]
  Save return value = 1.
```

```
  STEP 1 — Move last element (5) to root, remove last position:

       5         <- last element moved to root
     /   \
    3     2
   / \   /
  7   4 8

  Array: [5, 3, 2, 7, 4, 8]

  Compare 5 with children: left=3, right=2.
  Smaller child is 2 (index 2).  5 > 2, so swap.
```

```
  STEP 2 — Swap 5 and 2:

       2
     /   \
    3     5      <- 5 sifted down
   / \   /
  7   4 8

  Array: [2, 3, 5, 7, 4, 8]

  Compare 5 with children: left=8, right=none.
  Smaller child is 8 (index 5).  5 < 8, STOP.
```

```
  FINAL — Heap property restored, return 1:

       2
     /   \
    3     5
   / \   /
  7   4 8

  Array: [2, 3, 5, 7, 4, 8]
```

**Time**: O(log n) — at most we travel from the root to a leaf.

### Peek

Simply return `arr[0]`. The root always holds the min (or max).

**Time**: O(1).

### Build Heap (Heapify an Array)

Given an arbitrary array, rearrange it into a valid heap.

**Naive approach**: Insert elements one by one. Each insert is O(log n), so total is O(n log n).

**Optimal approach (Floyd's algorithm)**: Start from the last non-leaf node and sift down each node. This runs in **O(n)** — not O(n log n).

Why O(n)? Most nodes are near the bottom of the tree where sift-down is cheap. Half the nodes are leaves (0 work). A quarter of the nodes sift down at most 1 level. An eighth sift down at most 2 levels. The sum converges to O(n).

**Example**: Build a min-heap from `[8, 3, 7, 1, 5, 4, 2]`.

```
  STEP 0 — Interpret array as a complete binary tree (no heap property yet):

       8
     /   \
    3     7
   / \   / \
  1   5 4   2

  Last non-leaf index = (7/2) - 1 = 2.  Process indices 2, 1, 0.
```

```
  STEP 1 — Sift down index 2 (value 7):
  Children: 4, 2.  Smallest child = 2.  7 > 2, swap.

       8
     /   \
    3     2
   / \   / \
  1   5 4   7
```

```
  STEP 2 — Sift down index 1 (value 3):
  Children: 1, 5.  Smallest child = 1.  3 > 1, swap.

       8
     /   \
    1     2
   / \   / \
  3   5 4   7
```

```
  STEP 3 — Sift down index 0 (value 8):
  Children: 1, 2.  Smallest child = 1.  8 > 1, swap.

       1
     /   \
    8     2
   / \   / \
  3   5 4   7

  Continue sifting 8 down from index 1:
  Children: 3, 5.  Smallest child = 3.  8 > 3, swap.

       1
     /   \
    3     2
   / \   / \
  8   5 4   7

  Node 8 at index 3 has no children (indices 7,8 are out of bounds). STOP.
```

```
  FINAL — Valid min-heap:

       1
     /   \
    3     2
   / \   / \
  8   5 4   7

  Array: [1, 3, 2, 8, 5, 4, 7]
```

## 🔹 Complexity Table

| Operation       | Time Complexity | Notes                                          |
|-----------------|-----------------|-------------------------------------------------|
| Peek (find min) | O(1)            | Root is always the min/max                     |
| Insert          | O(log n)        | Bubble up at most tree height                  |
| Extract-Min/Max | O(log n)        | Sift down at most tree height                  |
| Build Heap      | O(n)            | Floyd's algorithm, not n x insert              |
| Search          | O(n)            | Heap is NOT ordered like a BST                 |
| Delete arbitrary| O(n)            | Must search first, then O(log n) to fix        |
| Decrease key    | O(log n)        | Update value, then sift up                     |
| Increase key    | O(log n)        | Update value, then sift down                   |

**Space**: O(n) for storing n elements.

## 🔹 Common Pitfalls

1. **Off-by-one in array indexing**: Using 1-based vs 0-based indexing changes the formulas. With 1-based: children at `2i` and `2i+1`, parent at `i/2`. With 0-based: children at `2i+1` and `2i+2`, parent at `(i-1)/2`. Pick one and be consistent.

2. **Confusing heap with BST**: A heap does NOT maintain sorted order among siblings. You cannot do an in-order traversal to get sorted output. Left child can be larger than right child — only the parent-child relationship matters.

3. **Using O(n log n) to build a heap**: Always use the bottom-up heapify approach (Floyd's algorithm) for O(n). Inserting elements one by one is slower.

4. **Forgetting to handle the sift-down edge case**: When sifting down, a node might have only a left child (no right child). Always check bounds before comparing the right child.

5. **Assuming extract gives you sorted order in one call**: Extract-min gives you the minimum, but the rest of the array is NOT sorted. To fully sort, you must extract n times (that is heapsort).

6. **Choosing min-heap vs max-heap**: Need the smallest? Min-heap. Need the largest? Max-heap. Need the K-th largest? Use a min-heap of size K (counterintuitive but correct — the root of a size-K min-heap is the K-th largest seen so far).

## 🔹 Template Code (C++ Min-Heap)

```cpp
#include <vector>
#include <stdexcept>

class MinHeap {
private:
    std::vector<int> data;

    int parent(int i) { return (i - 1) / 2; }
    int left(int i)   { return 2 * i + 1; }
    int right(int i)  { return 2 * i + 2; }

    // Bubble element at index i upward until heap property is restored
    void siftUp(int i) {
        while (i > 0 && data[i] < data[parent(i)]) {
            std::swap(data[i], data[parent(i)]);
            i = parent(i);
        }
    }

    // Push element at index i downward until heap property is restored
    void siftDown(int i) {
        int n = data.size();
        while (true) {
            int smallest = i;
            int l = left(i);
            int r = right(i);

            if (l < n && data[l] < data[smallest])
                smallest = l;
            if (r < n && data[r] < data[smallest])
                smallest = r;

            if (smallest == i) break;  // heap property satisfied

            std::swap(data[i], data[smallest]);
            i = smallest;
        }
    }

public:
    // Build heap from existing array — O(n)
    MinHeap() {}

    MinHeap(const std::vector<int>& arr) : data(arr) {
        // Floyd's algorithm: sift down from last non-leaf to root
        for (int i = (int)data.size() / 2 - 1; i >= 0; i--) {
            siftDown(i);
        }
    }

    // Insert element — O(log n)
    void insert(int val) {
        data.push_back(val);
        siftUp(data.size() - 1);
    }

    // View minimum without removing — O(1)
    int peek() const {
        if (data.empty()) throw std::runtime_error("Heap is empty");
        return data[0];
    }

    // Remove and return minimum — O(log n)
    int extractMin() {
        if (data.empty()) throw std::runtime_error("Heap is empty");

        int minVal = data[0];
        data[0] = data.back();
        data.pop_back();

        if (!data.empty()) {
            siftDown(0);
        }
        return minVal;
    }

    int size() const { return data.size(); }
    bool empty() const { return data.empty(); }
};
```

**Usage:**

```cpp
MinHeap heap;
heap.insert(5);
heap.insert(3);
heap.insert(8);
heap.insert(1);

// peek: 1
// extractMin: 1, then 3, then 5, then 8
```

## 🔹 Heapsort

Heapsort uses the heap structure to sort an array **in-place** with O(n log n) time and O(1) extra space.

**Algorithm** (ascending order using a max-heap):

1. Build a max-heap from the array — O(n).
2. Repeatedly swap the root (max) with the last unsorted element, shrink the heap by one, and sift down the new root — O(n log n) total.

```cpp
void heapsort(std::vector<int>& arr) {
    int n = arr.size();

    // Step 1: Build max-heap (sift down from last non-leaf)
    for (int i = n / 2 - 1; i >= 0; i--)
        siftDownMax(arr, n, i);  // max-heap sift down within size n

    // Step 2: Extract max repeatedly
    for (int i = n - 1; i > 0; i--) {
        std::swap(arr[0], arr[i]);    // move max to sorted region
        siftDownMax(arr, i, 0);       // restore heap in reduced range
    }
}
```

**Complexity**: O(n log n) worst-case, O(1) space. Not stable (does not preserve order of equal elements).

## 🔹 Related Topics

- [[Priority Queue]] — the primary abstraction built on top of heaps
- [[Binary Search Tree]] — ordered tree with different invariants; do not confuse with heap
- [[Sorting Algorithms]] — heapsort as an in-place O(n log n) sort
