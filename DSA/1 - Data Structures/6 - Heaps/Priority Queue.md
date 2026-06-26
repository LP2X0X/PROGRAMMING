---
tags:
  - algorithms
  - data-structure
  - heap
  - priority-queue
---

## 🔹 Real-World Analogy

**Emergency Room**: Patients are not seen in arrival order. A person having a heart attack gets treated before someone with a sprained ankle, regardless of who arrived first. Each patient is assigned a priority (triage level), and the highest-priority patient is always served next.

**OS Task Scheduler**: Your operating system runs dozens of processes, but the CPU can only execute one at a time (per core). The scheduler uses a priority queue to decide which process runs next — high-priority system interrupts preempt low-priority background tasks.

**Airport Boarding**: First class boards before economy. Within economy, families with small children board before general passengers. The gate agent maintains a mental priority queue.

## 🔹 What Is a Priority Queue?

A priority queue is an **abstract data type** (ADT) where each element has an associated **priority**, and the element with the highest priority is always served (dequeued) first.

It is NOT a specific data structure — it is a **contract** (interface). You can implement it with various backing structures, but a [[Min Heap and Max Heap|heap]] is the most efficient general-purpose choice.

Key distinction from a regular queue:

- **Regular queue (FIFO)**: first in, first out. Order of arrival determines service order.
- **Priority queue**: priority determines service order. Arrival order is irrelevant (or used only as a tiebreaker).

By convention:
- A **max-priority queue** serves the largest-priority element first.
- A **min-priority queue** serves the smallest-priority element first.

Either can be trivially converted to the other by negating priorities.

## 🔹 Why Use a Heap as the Backing Structure?

| Implementation         | Insert    | Extract Best | Peek   | Notes                              |
|------------------------|-----------|-------------|--------|------------------------------------|
| Unsorted array/list    | O(1)      | O(n)        | O(n)   | Fast insert, slow extract          |
| Sorted array/list      | O(n)      | O(1)        | O(1)   | Slow insert due to shifting        |
| Balanced BST           | O(log n)  | O(log n)    | O(log n)| Overkill — maintains full ordering |
| **Binary Heap**        | O(log n)  | O(log n)    | O(1)   | Best balance of all operations     |
| Fibonacci Heap         | O(1)*     | O(log n)*   | O(1)   | Amortized; complex to implement    |

The binary heap hits the sweet spot: O(log n) insert, O(log n) extract, O(1) peek, simple to implement, cache-friendly array layout, and no pointer overhead.

## 🔹 Operations and Complexities

| Operation              | Description                                     | Time      |
|------------------------|-------------------------------------------------|-----------|
| `insert(element, priority)` | Add element with given priority           | O(log n)  |
| `extractMax()` / `extractMin()` | Remove and return highest-priority element | O(log n) |
| `peek()`               | View highest-priority element without removing  | O(1)      |
| `changePriority(element, newPriority)` | Update an element's priority      | O(log n)  |
| `remove(element)`      | Remove a specific element                       | O(n)*     |
| `size()` / `empty()`   | Query the number of elements                    | O(1)      |

*O(n) for arbitrary removal because finding the element requires a linear scan. With an auxiliary hash map tracking positions, this can be reduced to O(log n).

## 🔹 Comparison: Priority Queue vs Queue vs Stack

```
  Regular Queue (FIFO):     Stack (LIFO):        Priority Queue:
  
  IN -> [D][C][B][A] -> OUT   TOP -> [D]           Highest priority -> OUT
                               [C]
  First in = first out         [B]            Elements ordered by priority,
                               [A]            NOT by insertion time
                              BOTTOM
```

| Feature           | Queue       | Stack       | Priority Queue         |
|-------------------|-------------|-------------|------------------------|
| Order             | FIFO        | LIFO        | By priority            |
| Insert            | O(1)        | O(1)        | O(log n) with heap     |
| Remove            | O(1)        | O(1)        | O(log n) with heap     |
| Peek              | O(1)        | O(1)        | O(1)                   |
| Use case          | BFS, buffer | DFS, undo   | Dijkstra, scheduling   |

## 🔹 Use Cases in Algorithms

### Dijkstra's Shortest Path

The priority queue holds `(distance, node)` pairs. At each step, extract the node with the smallest tentative distance, then relax its neighbors. Without a priority queue, you would need O(V) to find the minimum at each step, making the algorithm O(V^2). With a min-heap, Dijkstra runs in O((V + E) log V).

```
  Graph:              Priority Queue State:
  A --1-- B           Start: [(0,A)]
  |       |           Extract A(0): relax B(1), C(4)
  4       2           PQ: [(1,B), (4,C)]
  |       |           Extract B(1): relax D(3)
  C --5-- D           PQ: [(3,D), (4,C)]
                      Extract D(3): ...
```

### Huffman Coding

Build an optimal prefix code by repeatedly extracting the two lowest-frequency nodes and merging them. A min-priority queue makes each extraction O(log n).

### K-th Largest/Smallest Element

Maintain a min-heap of size K. After processing all elements, the root is the K-th largest. This runs in O(n log K) time and O(K) space — far better than sorting when K is much smaller than n.

### Merge K Sorted Lists

Push the head of each list into a min-heap. Extract the minimum, add it to the result, and push the next element from that list. Each of the n total elements enters and leaves the heap once: O(n log K).

### Task Scheduling

Given tasks with deadlines and penalties, use a max-priority queue to greedily schedule the highest-penalty tasks first, ensuring they meet their deadlines.

### Running Median (Two Heaps)

Maintain a max-heap for the lower half and a min-heap for the upper half. The median is always accessible from the tops of the two heaps. See the template code section below.

## 🔹 C++ STL `priority_queue`

The standard library provides `std::priority_queue` in `<queue>`.

**Critical gotcha**: `std::priority_queue` is a **MAX-heap** by default. The largest element is at the top.

```cpp
#include <queue>
#include <vector>

// MAX-heap (default) — largest element on top
std::priority_queue<int> maxPQ;
maxPQ.push(3);
maxPQ.push(1);
maxPQ.push(5);
// maxPQ.top() == 5

// MIN-heap — smallest element on top
// Template: priority_queue<Type, Container, Comparator>
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;
minPQ.push(3);
minPQ.push(1);
minPQ.push(5);
// minPQ.top() == 1
```

### STL Operations

| Method     | Description                              | Time     |
|------------|------------------------------------------|----------|
| `push(x)`  | Insert element                          | O(log n) |
| `pop()`    | Remove top element (does NOT return it) | O(log n) |
| `top()`    | Access top element without removing     | O(1)     |
| `empty()`  | Check if empty                          | O(1)     |
| `size()`   | Number of elements                      | O(1)     |

### Common STL Gotchas

1. **`pop()` returns void**: Unlike `extractMin()` in textbooks, STL separates access (`top()`) from removal (`pop()`). Always call `top()` before `pop()`.

2. **No decrease-key**: STL `priority_queue` does not support updating a key. The standard workaround is **lazy deletion**: insert a new entry with the updated priority and mark the old one as invalid. When you extract, skip entries that are stale.

3. **Custom comparator for structs**:

```cpp
struct Task {
    int priority;
    std::string name;
};

// For a min-heap of Tasks (lowest priority number on top):
auto cmp = [](const Task& a, const Task& b) {
    return a.priority > b.priority;  // NOTE: > for min-heap
};
std::priority_queue<Task, std::vector<Task>, decltype(cmp)> pq(cmp);
```

The comparator is **reversed** from what you might expect: return `true` if `a` should come AFTER `b`. Think of it as "a has lower priority than b."

4. **Pairs sort lexicographically**: `priority_queue<pair<int,int>>` sorts by `.first` first, then `.second`. For Dijkstra, use `pair<int,int>` as `(distance, node)` with `greater<>` for a min-heap:

```cpp
// Min-heap of (distance, node) for Dijkstra
std::priority_queue<
    std::pair<int,int>,
    std::vector<std::pair<int,int>>,
    std::greater<std::pair<int,int>>
> pq;
```

## 🔹 Template Code: Common Patterns

### Pattern 1 — Top K Elements

**Problem**: Find the K largest elements from a collection of n elements.

**Strategy**: Use a min-heap of size K. The root is the smallest of the K largest seen so far. If a new element is bigger than the root, evict the root and insert the new element.

```
  Maintaining a min-heap of size K=3 while scanning [5, 2, 9, 1, 7, 6, 8]:

  Process 5:  heap = [5]            (size < K, just add)
  Process 2:  heap = [2, 5]         (size < K, just add)
  Process 9:  heap = [2, 5, 9]      (size == K)
  Process 1:  1 < root 2, skip      (too small to be in top K)
  Process 7:  7 > root 2, replace   heap = [5, 7, 9]
  Process 6:  6 > root 5, replace   heap = [6, 7, 9]
  Process 8:  8 > root 6, replace   heap = [7, 8, 9]

  Result: top 3 = {7, 8, 9}
```

```cpp
std::vector<int> topK(const std::vector<int>& nums, int k) {
    // Min-heap — smallest of the top-K is at root
    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;

    for (int num : nums) {
        minHeap.push(num);
        if ((int)minHeap.size() > k) {
            minHeap.pop();  // evict the smallest — it's not in top K
        }
    }

    std::vector<int> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top());
        minHeap.pop();
    }
    return result;  // K largest elements (not necessarily sorted)
}
// Time: O(n log K)   Space: O(K)
```

For the K **smallest** elements, use a max-heap of size K and evict the largest when size exceeds K.

### Pattern 2 — Merge K Sorted Lists

**Problem**: Given K sorted lists (or linked lists), merge them into a single sorted list.

**Strategy**: Push the first element from each list into a min-heap. Extract the minimum, append it to the result, and push the next element from that same list.

```
  Lists:  [1, 4, 7]    [2, 5, 8]    [3, 6, 9]

  Heap:         Extract    Push next    Result so far
  [1, 2, 3]  ->  1     ->  push 4   ->  [1]
  [2, 3, 4]  ->  2     ->  push 5   ->  [1, 2]
  [3, 4, 5]  ->  3     ->  push 6   ->  [1, 2, 3]
  [4, 5, 6]  ->  4     ->  push 7   ->  [1, 2, 3, 4]
  ...and so on until all lists are exhausted.
```

```cpp
struct Element {
    int val;
    int listIdx;  // which list this came from
    int elemIdx;  // index within that list
};

auto cmp = [](const Element& a, const Element& b) {
    return a.val > b.val;  // min-heap
};

std::vector<int> mergeKSorted(const std::vector<std::vector<int>>& lists) {
    std::priority_queue<Element, std::vector<Element>, decltype(cmp)> pq(cmp);

    // Push the first element from each list
    for (int i = 0; i < (int)lists.size(); i++) {
        if (!lists[i].empty()) {
            pq.push({lists[i][0], i, 0});
        }
    }

    std::vector<int> result;
    while (!pq.empty()) {
        auto [val, li, ei] = pq.top();
        pq.pop();
        result.push_back(val);

        // Push next element from the same list
        if (ei + 1 < (int)lists[li].size()) {
            pq.push({lists[li][ei + 1], li, ei + 1});
        }
    }
    return result;
}
// Time: O(N log K) where N = total elements across all lists
// Space: O(K) for the heap
```

### Pattern 3 — Running Median (Two Heaps)

**Problem**: Given a stream of numbers, efficiently find the median after each insertion.

**Strategy**: Maintain two heaps that split the data into a lower half and an upper half:
- **Max-heap** (`lo`): stores the smaller half. Its top is the largest of the small elements.
- **Min-heap** (`hi`): stores the larger half. Its top is the smallest of the large elements.

Balance them so that `lo.size()` is either equal to or one greater than `hi.size()`. The median is then `lo.top()` (odd count) or the average of both tops (even count).

```
  Stream: 5, 2, 8, 1, 7

  After 5:   lo=[5]       hi=[]        median = 5
  After 2:   lo=[2]       hi=[5]       median = (2+5)/2 = 3.5
  After 8:   lo=[5,2]     hi=[8]       median = 5
  After 1:   lo=[2,1]     hi=[5,8]     median = (2+5)/2 = 3.5
  After 7:   lo=[5,2,1]   hi=[7,8]     median = 5

  Visualization of the two heaps:

    lo (max-heap)    |    hi (min-heap)
                     |
      top -> 5       |       5 <- top
            / \      |      /
           2   1     |     8
                     |
    "small half"     |    "large half"
    (top = largest   |    (top = smallest
     of the smalls)  |     of the larges)
```

```cpp
class MedianFinder {
    // Max-heap for lower half
    std::priority_queue<int> lo;
    // Min-heap for upper half
    std::priority_queue<int, std::vector<int>, std::greater<int>> hi;

public:
    void addNum(int num) {
        // Step 1: Add to appropriate heap
        if (lo.empty() || num <= lo.top()) {
            lo.push(num);
        } else {
            hi.push(num);
        }

        // Step 2: Rebalance — lo can have at most 1 more element than hi
        if ((int)lo.size() > (int)hi.size() + 1) {
            hi.push(lo.top());
            lo.pop();
        } else if ((int)hi.size() > (int)lo.size()) {
            lo.push(hi.top());
            hi.pop();
        }
    }

    double findMedian() {
        if (lo.size() > hi.size()) {
            return lo.top();  // odd total count
        }
        return (lo.top() + hi.top()) / 2.0;  // even total count
    }
};
// Insert: O(log n)    FindMedian: O(1)
```

## 🔹 Common Pitfalls

1. **Forgetting STL priority_queue is a max-heap**: The default behavior gives you the largest element. For algorithms like Dijkstra that need the smallest, you must use `greater<>`.

2. **Calling `pop()` and expecting a return value**: STL `pop()` returns `void`. Always read `top()` first.

3. **No built-in decrease-key in STL**: If you need to update priorities (e.g., in Dijkstra), use lazy deletion or switch to a custom heap with a position map.

4. **Confusing the comparator direction**: For `std::priority_queue`, the comparator returns `true` if the first argument should be placed BELOW the second. So `greater<int>` gives a min-heap (counterintuitive at first).

5. **Using a priority queue where a simple sort suffices**: If you process all elements at once and just need them sorted, `std::sort` is simpler and often faster. Priority queues shine when elements arrive incrementally or you only need the top K.

6. **Choosing the wrong heap type for Top-K**: For the K **largest**, use a min-heap of size K (not a max-heap). For the K **smallest**, use a max-heap of size K. The heap holds the "boundary" elements, and the root is the one you might evict.

## 🔹 Related Topics

- [[Min Heap and Max Heap]] — the underlying data structure that powers priority queues
- [[Graph Algorithms]] — Dijkstra and Prim rely heavily on priority queues
- [[Sorting Algorithms]] — heapsort uses the heap extraction pattern
