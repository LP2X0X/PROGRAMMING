---
tags:
  - algorithms
  - data-structure
  - queue
---

## 🔹 What Is a Queue?

A queue is a linear data structure that follows **FIFO** (First In, First Out) ordering. The first element added is the first one removed -- just like a line at a store.

**Real-world analogies:**
- People waiting in line at a checkout counter
- A ticket queue at a help desk
- Cars lined up at a drive-through
- Print jobs waiting for a printer

```
  ENQUEUE (add to back)                    DEQUEUE (remove from front)
         |                                        |
         v                                        v
  +------+------+------+------+------+------+------+
  | Back |  E   |  D   |  C   |  B   |  A   | Front|
  +------+------+------+------+------+------+------+
         <-- items wait in order, move toward front -->

  New items enter at the BACK.
  Items leave from the FRONT.
```

The key insight: elements are processed in exactly the order they arrive. This property makes queues essential for **fairness** and **level-by-level exploration** (BFS).

---

## 🔹 Core Operations

All fundamental queue operations run in **O(1)** time:

| Operation      | Description                          | Time |
| -------------- | ------------------------------------ | ---- |
| `enqueue(x)`   | Add item `x` to the back (rear)     | O(1) |
| `dequeue()`    | Remove and return the front item     | O(1) |
| `peek()`       | View the front item without removal  | O(1) |
| `isEmpty()`    | Return `true` if queue has no items  | O(1) |
| `size()`       | Return the number of items in queue  | O(1) |

```cpp
// Basic interface
void  enqueue(T item);   // add to back
T     dequeue();          // remove from front
T     peek();             // view front (also called front())
bool  isEmpty();          // check if empty
int   size();             // element count
```

---

## 🔹 Implementation: Circular Array (Ring Buffer)

### The Problem with a Naive Array

If you use a plain array and always dequeue from index 0, you must shift every remaining element left -- that is O(n) per dequeue. Even if you avoid shifting by advancing a `front` pointer, you waste all the space behind `front` that can never be reused:

```
  After several enqueues and dequeues with a naive front pointer:

  Index:   0     1     2     3     4     5     6     7
         [   ] [   ] [   ] [ C ] [ D ] [ E ] [   ] [   ]
                              ^                 ^
                            front              rear

  Indices 0-2 are permanently wasted.
```

### The Circular Solution

A **circular array** (ring buffer) wraps around using modulo arithmetic. When `rear` reaches the end, it wraps to index 0, reusing the vacated space:

```
  Circular array (capacity = 8), currently holding C, D, E, F:

        +-----+-----+-----+-----+-----+-----+-----+-----+
  Index |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |
        +-----+-----+-----+-----+-----+-----+-----+-----+
  Data  |     |     |     |  C  |  D  |  E  |  F  |     |
        +-----+-----+-----+-----+-----+-----+-----+-----+
                              ^                       ^
                            front                   rear

  After enqueue(G), enqueue(H), enqueue(I):

        +-----+-----+-----+-----+-----+-----+-----+-----+
  Index |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |
        +-----+-----+-----+-----+-----+-----+-----+-----+
  Data  |  I  |     |     |  C  |  D  |  E  |  F  |  G  |
        +-----+-----+-----+-----+-----+-----+-----+-----+
                ^              ^
              rear           front

  rear WRAPPED AROUND from index 7 -> 0 via (rear + 1) % capacity.
```

### Full vs Empty Ambiguity

Both "full" and "empty" can result in `front == rear`. Two common solutions:

1. **Track a `count` variable** (simplest): full when `count == capacity`, empty when `count == 0`.
2. **Waste one slot**: keep one slot always empty. Full when `(rear + 1) % capacity == front`. Empty when `front == rear`.

### Implementation (using count)

```cpp
class CircularQueue {
    int* data;
    int front, rear, count, capacity;

public:
    CircularQueue(int cap) : capacity(cap), front(0), rear(0), count(0) {
        data = new int[capacity];
    }

    void enqueue(int item) {
        if (count == capacity) throw "Queue is full";
        data[rear] = item;
        rear = (rear + 1) % capacity;   // wrap around
        count++;
    }

    int dequeue() {
        if (count == 0) throw "Queue is empty";
        int item = data[front];
        front = (front + 1) % capacity;  // wrap around
        count--;
        return item;
    }

    int peek()    { return data[front]; }
    bool isEmpty() { return count == 0; }
    int size()     { return count; }

    ~CircularQueue() { delete[] data; }
};
```

---

## 🔹 Implementation: Linked List

Use a singly linked list with both **head** (front) and **tail** (rear) pointers. Enqueue appends at tail, dequeue removes from head -- both O(1).

```
  Enqueue at tail, dequeue from head:

  head                                          tail
   |                                             |
   v                                             v
  [A] --> [B] --> [C] --> [D] --> [E] --> [F] --> null
   ^                                       ^
   |                                       |
  dequeue()                           enqueue()
  removes A                           adds here
```

### Implementation

```cpp
struct Node {
    int data;
    Node* next;
    Node(int d) : data(d), next(nullptr) {}
};

class LinkedQueue {
    Node* head;   // front -- dequeue from here
    Node* tail;   // rear  -- enqueue here
    int count;

public:
    LinkedQueue() : head(nullptr), tail(nullptr), count(0) {}

    void enqueue(int item) {
        Node* node = new Node(item);
        if (tail) tail->next = node;
        else      head = node;          // first element
        tail = node;
        count++;
    }

    int dequeue() {
        if (!head) throw "Queue is empty";
        int item = head->data;
        Node* old = head;
        head = head->next;
        if (!head) tail = nullptr;      // queue is now empty
        delete old;
        count--;
        return item;
    }

    int peek()     { return head->data; }
    bool isEmpty() { return head == nullptr; }
    int size()     { return count; }

    ~LinkedQueue() {
        while (head) {
            Node* tmp = head;
            head = head->next;
            delete tmp;
        }
    }
};
```

---

## 🔹 Implementation Comparison

| Aspect           | Circular Array                        | Linked List                           |
| ---------------- | ------------------------------------- | ------------------------------------- |
| Memory layout    | Contiguous, cache-friendly            | Scattered nodes, more cache misses    |
| Memory overhead  | Fixed allocation, may waste if sparse | Per-node pointer overhead (~8 bytes)  |
| Resize           | Must allocate + copy (amortized O(1)) | Grows naturally, no resize needed     |
| Max capacity     | Fixed unless you implement resizing   | Limited only by available memory      |
| Cache perf       | Excellent (spatial locality)          | Poor (pointer chasing)                |
| Implementation   | Slightly tricky (modulo, full/empty)  | Straightforward with head/tail        |
| Best for         | Known max size, performance-critical  | Unknown size, frequent grow/shrink    |

In practice, most standard library queues (C++ `std::queue`, Java `ArrayDeque`) use a circular array or dynamic array internally because cache performance dominates for typical workloads.

---

## 🔹 Common Uses in Algorithms

### BFS (Breadth-First Search) -- THE defining use case

Queues enable **level-by-level exploration** of graphs and trees. BFS discovers all nodes at distance `d` before any node at distance `d+1`. This works because the queue processes nodes in the exact order they were discovered -- FIFO guarantees this layered expansion.

See the detailed BFS walkthrough below.

### Level-Order Traversal of Binary Trees

Visit all nodes level by level, left to right. This is BFS applied to a [[Trees|tree]].

### Task Scheduling

- **Round-robin scheduling**: processes take turns in queue order
- **Print queues**: documents printed in submission order
- **Job/task scheduling**: batch processing systems

### Buffering

- **I/O buffers**: keyboard input buffer, network packet queues
- **Producer-consumer pattern**: producers enqueue work, consumers dequeue it
- **Message queues**: inter-process or inter-service communication (e.g., RabbitMQ, Kafka)

### Sliding Window Problems

Many sliding window problems use a [[Deque]] (double-ended queue) to maintain a window of candidates. The deque extends the queue concept by allowing insertion/removal at both ends.

---

## 🔹 BFS with Queue -- Step-by-Step Walkthrough

BFS is the single most important reason queues exist in algorithm design. Understanding this thoroughly is essential.

### The Graph

```
        0
       / \
      1    2
     / \    \
    3    4    5
         \
          6
```

Adjacency list:
```
0: [1, 2]
1: [0, 3, 4]
2: [0, 5]
3: [1]
4: [1, 6]
5: [2]
6: [4]
```

### BFS from node 0 -- step by step

```
Step 0: Start at node 0
  Mark 0 as visited, enqueue 0.
  Queue:  [ 0 ]
  Visited: {0}

Step 1: Dequeue 0. Process it. Enqueue unvisited neighbors 1, 2.
  Queue:  [ 1, 2 ]
  Visited: {0, 1, 2}
  Output: 0

Step 2: Dequeue 1. Process it. Enqueue unvisited neighbors 3, 4.
  (Neighbor 0 already visited -- skip.)
  Queue:  [ 2, 3, 4 ]
  Visited: {0, 1, 2, 3, 4}
  Output: 0 -> 1

Step 3: Dequeue 2. Process it. Enqueue unvisited neighbor 5.
  (Neighbor 0 already visited -- skip.)
  Queue:  [ 3, 4, 5 ]
  Visited: {0, 1, 2, 3, 4, 5}
  Output: 0 -> 1 -> 2

Step 4: Dequeue 3. Process it. No unvisited neighbors.
  (Neighbor 1 already visited -- skip.)
  Queue:  [ 4, 5 ]
  Visited: {0, 1, 2, 3, 4, 5}
  Output: 0 -> 1 -> 2 -> 3

Step 5: Dequeue 4. Process it. Enqueue unvisited neighbor 6.
  (Neighbor 1 already visited -- skip.)
  Queue:  [ 5, 6 ]
  Visited: {0, 1, 2, 3, 4, 5, 6}
  Output: 0 -> 1 -> 2 -> 3 -> 4

Step 6: Dequeue 5. Process it. No unvisited neighbors.
  Queue:  [ 6 ]
  Visited: {0, 1, 2, 3, 4, 5, 6}
  Output: 0 -> 1 -> 2 -> 3 -> 4 -> 5

Step 7: Dequeue 6. Process it. No unvisited neighbors.
  Queue:  [ ]     <-- empty, BFS complete
  Visited: {0, 1, 2, 3, 4, 5, 6}
  Output: 0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6
```

Notice the output order: `0, 1, 2, 3, 4, 5, 6`. Nodes are processed level by level:
- Level 0: `0`
- Level 1: `1, 2`
- Level 2: `3, 4, 5`
- Level 3: `6`

This is exactly what the FIFO property guarantees.

### Why BFS needs a queue (not a stack)

If you replaced the queue with a [[Stack]], you would get DFS (Depth-First Search) instead. The stack's LIFO order means the most recently discovered node is explored first, diving deep before going wide. The queue's FIFO order ensures the earliest discovered nodes are explored first, spreading wide before going deep.

---

## 🔹 Level-Order Tree Traversal

### The Tree

```
            1
          /   \
        2       3
       / \       \
      4   5       6
         / \
        7   8
```

### Queue state at each level

```
Start: enqueue root (1)
  Queue: [ 1 ]

Level 0 -- process 1 node:
  Dequeue 1. Enqueue children 2, 3.
  Queue: [ 2, 3 ]
  Output: [1]

Level 1 -- process 2 nodes:
  Dequeue 2. Enqueue children 4, 5.
  Dequeue 3. Enqueue child 6.
  Queue: [ 4, 5, 6 ]
  Output: [1] [2, 3]

Level 2 -- process 3 nodes:
  Dequeue 4. No children.
  Dequeue 5. Enqueue children 7, 8.
  Dequeue 6. No children.
  Queue: [ 7, 8 ]
  Output: [1] [2, 3] [4, 5, 6]

Level 3 -- process 2 nodes:
  Dequeue 7. No children.
  Dequeue 8. No children.
  Queue: [ ]    <-- empty, traversal complete
  Output: [1] [2, 3] [4, 5, 6] [7, 8]
```

The trick for level-by-level processing: at the start of each level, record `levelSize = queue.size()`, then dequeue exactly that many nodes. Everything enqueued during that loop belongs to the next level.

---

## 🔹 Time and Space Complexity

| Operation  | Circular Array | Linked List | Notes                          |
| ---------- | -------------- | ----------- | ------------------------------ |
| enqueue    | O(1)*          | O(1)        | *Amortized if array resizes    |
| dequeue    | O(1)           | O(1)        |                                |
| peek       | O(1)           | O(1)        |                                |
| isEmpty    | O(1)           | O(1)        |                                |
| size       | O(1)           | O(1)        |                                |
| Space      | O(n)           | O(n)        | Linked list has pointer overhead |

| BFS Algorithm    | Time     | Space  | Notes                                      |
| ---------------- | -------- | ------ | ------------------------------------------ |
| Graph BFS        | O(V + E) | O(V)   | V = vertices, E = edges; queue holds up to V nodes |
| Tree level-order | O(n)     | O(w)   | w = max width of tree (widest level)       |

---

## 🔹 Common Pitfalls

**1. Forgetting to mark nodes as visited in BFS**
If you enqueue a neighbor without immediately marking it visited, other nodes may also enqueue that same neighbor. This leads to duplicate processing and, in cyclic graphs, an infinite loop. Always mark visited **when enqueuing**, not when dequeuing.

```cpp
// WRONG: mark when dequeuing
queue.enqueue(neighbor);
// ... later ...
visited[node] = true;   // too late -- neighbor may be enqueued multiple times

// CORRECT: mark when enqueuing
if (!visited[neighbor]) {
    visited[neighbor] = true;   // mark immediately
    queue.enqueue(neighbor);
}
```

**2. Array-based queue without circular logic**
Dequeuing from the front of a non-circular array either requires O(n) shifting or wastes O(n) space from abandoned front slots. Always use a circular (ring buffer) approach for array-based queues.

**3. Confusing queue with stack**
- Queue = FIFO = BFS (level-by-level)
- [[Stack]] = LIFO = DFS (depth-first)

Mixing them up in BFS/DFS implementations is a classic mistake.

**4. Off-by-one in circular array**
The modulo arithmetic `(index + 1) % capacity` is easy to get wrong when handling full/empty states. Using a separate `count` variable is the safest approach.

---

## 🔹 Template Code

### BFS on a Graph (Adjacency List)

```cpp
#include <vector>
#include <queue>

// BFS from a source node in an unweighted graph.
// Returns the BFS visit order (can be adapted for shortest path, etc.)
vector<int> bfs(int source, const vector<vector<int>>& adj, int n) {
    vector<bool> visited(n, false);
    vector<int> order;           // BFS visit order
    queue<int> q;

    // Start: mark source visited and enqueue it
    visited[source] = true;
    q.push(source);

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        order.push_back(node);   // "process" this node

        // Explore all neighbors
        for (int neighbor : adj[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;   // mark BEFORE enqueuing
                q.push(neighbor);
            }
        }
    }

    return order;
}

// Usage:
// int n = 7;  // number of nodes (0 to 6)
// vector<vector<int>> adj(n);
// adj[0] = {1, 2};
// adj[1] = {0, 3, 4};
// adj[2] = {0, 5};
// adj[3] = {1};
// adj[4] = {1, 6};
// adj[5] = {2};
// adj[6] = {4};
// vector<int> result = bfs(0, adj, n);
// result: [0, 1, 2, 3, 4, 5, 6]
```

### BFS Shortest Path (Unweighted Graph)

```cpp
#include <vector>
#include <queue>

// Find shortest distance from source to all reachable nodes.
// Returns dist[] where dist[i] = shortest distance from source to i.
// dist[i] = -1 if i is unreachable.
vector<int> bfsShortestPath(int source, const vector<vector<int>>& adj, int n) {
    vector<int> dist(n, -1);     // -1 means unvisited/unreachable
    queue<int> q;

    dist[source] = 0;
    q.push(source);

    while (!q.empty()) {
        int node = q.front();
        q.pop();

        for (int neighbor : adj[node]) {
            if (dist[neighbor] == -1) {        // unvisited
                dist[neighbor] = dist[node] + 1;
                q.push(neighbor);
            }
        }
    }

    return dist;
}
```

### Level-Order Traversal of a Binary Tree

```cpp
#include <vector>
#include <queue>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int v) : val(v), left(nullptr), right(nullptr) {}
};

// Returns a 2D vector where result[i] contains all values at level i.
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int levelSize = q.size();       // number of nodes at this level
        vector<int> currentLevel;

        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front();
            q.pop();
            currentLevel.push_back(node->val);

            // Enqueue children for the next level
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }

        result.push_back(currentLevel);
    }

    return result;
}

// For the tree:
//         1
//       /   \
//     2       3
//    / \       \
//   4   5       6
//      / \
//     7   8
//
// Output: [[1], [2, 3], [4, 5, 6], [7, 8]]
```

---

## 🔹 Related Concepts

- [[Stack]] -- LIFO counterpart; stack is to DFS as queue is to BFS
- [[Deque]] -- double-ended queue; supports insert/remove at both ends; used in sliding window problems
- [[Arrays]] -- underlying storage for circular array implementation
- [[Singly Linked List]] -- underlying storage for linked list implementation
- [[Trees]] -- level-order traversal is BFS on a tree
- [[Graphs]] -- BFS is a fundamental graph traversal algorithm
- [[Priority Queue]] -- elements dequeued by priority, not arrival order; used in Dijkstra's algorithm
