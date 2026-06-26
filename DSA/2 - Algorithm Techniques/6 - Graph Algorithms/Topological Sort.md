---
tags:
  - algorithms
  - graph
  - topological-sort
related:
  - "[[BFS - Breadth First Search]]"
  - "[[DFS - Depth First Search]]"
  - "[[Graph Representations]]"
---

## 🔹 Real-World Analogy

**Course Prerequisites**: You must take Calculus I before Calculus II, and Linear Algebra
before Machine Learning. Topological sort gives you a **valid order** to take all courses
so that you never sit in a class without having completed its prerequisites.

**Build Systems**: In a Makefile or package manager, source files have dependencies.
You must compile `utils.o` before `main.o` if `main.cpp` includes `utils.h`.
Topological sort determines the correct **compilation order** so every dependency
is built before anything that needs it.

```
  Both scenarios have the same structure:

  +---------------------------------------------------------+
  |  A set of tasks with "must come before" relationships   |
  |  → Need a linear ordering that respects ALL of them     |
  +---------------------------------------------------------+
```

---

## 🔹 What Is Topological Sort?

A **linear ordering** of vertices in a **Directed Acyclic Graph (DAG)** such that
for every directed edge `u -> v`, vertex `u` comes **before** vertex `v` in the ordering.

```
  Critical properties:
  +-------------------------------------------------+
  | 1. Only works on DAGs (Directed Acyclic Graphs) |
  | 2. A DAG can have MULTIPLE valid orderings      |
  | 3. If the graph has a cycle, NO ordering exists  |
  +-------------------------------------------------+
```

> A topological ordering is essentially a way to "flatten" a DAG into a sequence
> while preserving all the dependency relationships.

```
  DAG:                           Flattened:
       A ──→ B ──→ D
       │           ↑             A, C, B, D   ✓
       └───→ C ────┘             A, B, C, D   ✓

  NOT a DAG (has cycle):         No valid ordering!
       A ──→ B ──→ C ──→ A      Who goes first? Nobody can.
```

---

## 🔹 ASCII Walkthrough — Course Prerequisite Graph

Consider this course dependency graph (arrow means "is prerequisite for"):

```
    CS101 ──→ CS201 ──→ CS301
      │         │
      ↓         ↓
    MATH ──→ CS202 ──→ CS401
```

Edges:
- CS101 → CS201,  CS101 → MATH
- CS201 → CS301,  CS201 → CS202
- MATH  → CS202
- CS202 → CS401

**Valid ordering 1**: CS101, MATH, CS201, CS202, CS301, CS401
**Valid ordering 2**: CS101, CS201, MATH, CS202, CS301, CS401
**Valid ordering 3**: CS101, CS201, CS301, MATH, CS202, CS401

Multiple valid orderings exist because some courses are independent of each other
(e.g., CS301 and MATH have no dependency between them).

```
  Invalid ordering: CS201, CS101, ...
      ✗  CS101 must come before CS201!

  Invalid ordering: CS202, MATH, ...
      ✗  MATH must come before CS202!
```

---

## 🔹 Approach 1 — Kahn's Algorithm (BFS-Based, In-Degree)

Uses [[BFS - Breadth First Search]] with an **in-degree** counter.

**Core idea**: Nodes with in-degree 0 have no prerequisites -- they can go first.
After "removing" them, new nodes may reach in-degree 0. Peel off layer by layer.

### Algorithm Steps

```
  1. Calculate in-degree for every vertex
  2. Push all vertices with in-degree 0 into a queue
  3. While queue is not empty:
     a. Dequeue vertex u, add to result
     b. For each neighbor v of u:
        - Decrement inDegree[v]
        - If inDegree[v] == 0, enqueue v
  4. If result.size() != n  →  CYCLE EXISTS!
```

### Detailed Step-by-Step Walkthrough

Using a numbered graph for clarity:

```
  Mapping:
  0: CS101    1: CS201    2: CS301
  3: MATH     4: CS202    5: CS401

  Adjacency list:
  0 → [1, 3]       (CS101 → CS201, CS101 → MATH)
  1 → [2, 4]       (CS201 → CS301, CS201 → CS202)
  2 → []
  3 → [4]           (MATH → CS202)
  4 → [5]           (CS202 → CS401)
  5 → []

  Graph visualization:
       0 ───→ 1 ───→ 2
       │      │
       ↓      ↓
       3 ───→ 4 ───→ 5
```

---

#### Initial State

```
  Vertex:     0     1     2     3     4     5
  In-deg:   [ 0 ] [ 1 ] [ 1 ] [ 1 ] [ 2 ] [ 1 ]
              ^     ^     ^     ^    ^ ^     ^
              |     |     |     |    | |     |
             none  from  from  from from   from
                    0     1     0   1  3     4

  Queue:  [ 0 ]          (only vertex 0 has in-degree 0)
  Result: [ ]
```

---

#### Step 1: Process vertex 0 (CS101)

```
  Dequeue: 0
  Neighbors of 0: {1, 3}
    inDegree[1]: 1 → 0  ✓ enqueue 1
    inDegree[3]: 1 → 0  ✓ enqueue 3

  Vertex:     0     1     2     3     4     5
  In-deg:   [ X ] [ 0 ] [ 1 ] [ 0 ] [ 2 ] [ 1 ]

  Queue:  [ 1, 3 ]
  Result: [ 0 ]

       X ···→ 1 ───→ 2        ("X" = processed)
              │
              ↓
       3 ───→ 4 ───→ 5
```

---

#### Step 2: Process vertex 1 (CS201)

```
  Dequeue: 1
  Neighbors of 1: {2, 4}
    inDegree[2]: 1 → 0  ✓ enqueue 2
    inDegree[4]: 2 → 1    (not 0 yet, don't enqueue)

  Vertex:     0     1     2     3     4     5
  In-deg:   [ X ] [ X ] [ 0 ] [ 0 ] [ 1 ] [ 1 ]

  Queue:  [ 3, 2 ]
  Result: [ 0, 1 ]

       X ···→ X ···→ 2
              ·
              ↓
       3 ───→ 4 ───→ 5
```

---

#### Step 3: Process vertex 3 (MATH)

```
  Dequeue: 3
  Neighbors of 3: {4}
    inDegree[4]: 1 → 0  ✓ enqueue 4

  Vertex:     0     1     2     3     4     5
  In-deg:   [ X ] [ X ] [ 0 ] [ X ] [ 0 ] [ 1 ]

  Queue:  [ 2, 4 ]
  Result: [ 0, 1, 3 ]

       X ···→ X ···→ 2

       X ···→ 4 ───→ 5
```

---

#### Step 4: Process vertex 2 (CS301)

```
  Dequeue: 2
  Neighbors of 2: {}    (no outgoing edges — leaf node)

  Vertex:     0     1     2     3     4     5
  In-deg:   [ X ] [ X ] [ X ] [ X ] [ 0 ] [ 1 ]

  Queue:  [ 4 ]
  Result: [ 0, 1, 3, 2 ]
```

---

#### Step 5: Process vertex 4 (CS202)

```
  Dequeue: 4
  Neighbors of 4: {5}
    inDegree[5]: 1 → 0  ✓ enqueue 5

  Vertex:     0     1     2     3     4     5
  In-deg:   [ X ] [ X ] [ X ] [ X ] [ X ] [ 0 ]

  Queue:  [ 5 ]
  Result: [ 0, 1, 3, 2, 4 ]
```

---

#### Step 6: Process vertex 5 (CS401)

```
  Dequeue: 5
  Neighbors of 5: {}    (leaf node)

  Queue:  [ ]  (empty!)
  Result: [ 0, 1, 3, 2, 4, 5 ]

  Result size (6) == n (6)  →  No cycle!  Valid topological order!

  Final: CS101 → CS201 → MATH → CS301 → CS202 → CS401  ✓
```

---

#### Summary Table — In-Degree at Each Step

```
  Step │ Process │ In-degree array          │ Queue after   │ Result so far
  ─────┼─────────┼──────────────────────────┼───────────────┼──────────────────
   0   │  init   │ [0, 1, 1, 1, 2, 1]      │ [0]           │ []
   1   │  v=0    │ [X, 0, 1, 0, 2, 1]      │ [1, 3]        │ [0]
   2   │  v=1    │ [X, X, 0, 0, 1, 1]      │ [3, 2]        │ [0, 1]
   3   │  v=3    │ [X, X, 0, X, 0, 1]      │ [2, 4]        │ [0, 1, 3]
   4   │  v=2    │ [X, X, X, X, 0, 1]      │ [4]           │ [0, 1, 3, 2]
   5   │  v=4    │ [X, X, X, X, X, 0]      │ [5]           │ [0, 1, 3, 2, 4]
   6   │  v=5    │ [X, X, X, X, X, X]      │ []            │ [0, 1, 3, 2, 4, 5]
```

---

## 🔹 Approach 2 — DFS-Based (Reverse Post-Order)

Uses [[DFS - Depth First Search]] with a **stack** (or reverse at the end).

**Core idea**: In DFS, a node is finished (post-order) only after ALL its descendants
are finished. So finished-last nodes should come first -- hence **reverse** post-order.

### Algorithm Steps

```
  1. Mark all nodes as unvisited (white / state=0)
  2. For each unvisited node, run DFS:
     a. Mark node as "visiting" (gray / state=1)
     b. Recurse on all unvisited neighbors
     c. If any neighbor is "visiting" → CYCLE!
     d. Mark node as "done" (black / state=2)
     e. Push node to result list
  3. Reverse the result list → topological order
```

### Three-Color State System

```
  +-------------------+------------------------------------------+
  | State 0 (White)   | Unvisited — not yet reached by DFS       |
  +-------------------+------------------------------------------+
  | State 1 (Gray)    | Visiting — currently in the DFS stack     |
  |                   | (on the path from root to current node)  |
  +-------------------+------------------------------------------+
  | State 2 (Black)   | Done — fully explored, all descendants   |
  |                   | have been processed                      |
  +-------------------+------------------------------------------+

  Cycle detection: If during DFS you reach a GRAY node,
  you've found a back edge → CYCLE!

  Why? A gray node is an ancestor in the current DFS path.
  Reaching it again means there's a path from it back to itself.
```

### Detailed DFS Walkthrough

Same graph:

```
       0 ───→ 1 ───→ 2
       │      │
       ↓      ↓
       3 ───→ 4 ───→ 5
```

```
  Start DFS from node 0:

  dfs(0)                          state: [G, W, W, W, W, W]
  │
  ├── dfs(1)                      state: [G, G, W, W, W, W]
  │   │
  │   ├── dfs(2)                  state: [G, G, G, W, W, W]
  │   │   └── no neighbors
  │   │   └── mark 2 BLACK        state: [G, G, B, W, W, W]
  │   │   └── push 2              order: [2]
  │   │
  │   ├── dfs(4)                  state: [G, G, B, W, G, W]
  │   │   │
  │   │   └── dfs(5)              state: [G, G, B, W, G, G]
  │   │       └── no neighbors
  │   │       └── mark 5 BLACK    state: [G, G, B, W, G, B]
  │   │       └── push 5          order: [2, 5]
  │   │
  │   │   └── mark 4 BLACK        state: [G, G, B, W, B, B]
  │   │   └── push 4              order: [2, 5, 4]
  │   │
  │   └── mark 1 BLACK            state: [G, B, B, W, B, B]
  │   └── push 1                  order: [2, 5, 4, 1]
  │
  ├── dfs(3)                      state: [G, B, B, G, B, B]
  │   ├── neighbor 4 is BLACK     (skip — already done)
  │   └── mark 3 BLACK            state: [G, B, B, B, B, B]
  │   └── push 3                  order: [2, 5, 4, 1, 3]
  │
  └── mark 0 BLACK                state: [B, B, B, B, B, B]
  └── push 0                      order: [2, 5, 4, 1, 3, 0]

  Reverse: [0, 3, 1, 4, 5, 2]

  Topological order: CS101, MATH, CS201, CS202, CS401, CS301  ✓
```

Notice this is a **different** valid ordering than Kahn's produced -- both are correct!

---

## 🔹 Cycle Detection — What It Looks Like

If we add an edge `5 → 1` creating a cycle:

```
       0 ───→ 1 ───→ 2
       │      ↑ │
       ↓      │ ↓
       3 ───→ 4 ←── 5
              └───→ 5 ──→ 1

  Cycle: 1 → 4 → 5 → 1

  ─── DFS detection ───────────────────────────────────────────
  dfs(0) → dfs(1) → dfs(4) → dfs(5)
    5's neighbor is 1, and state[1] == GRAY (visiting)
    → Back edge found → CYCLE!

  ─── Kahn's detection ────────────────────────────────────────
  In-degrees with the extra edge 5→1:
    0:0  1:2  2:1  3:1  4:2  5:1

  Only node 0 starts in queue. After processing 0:
    inDeg[1]=1, inDeg[3]=0. Process 3: inDeg[4]=1.
    Now queue is empty but only 3 nodes processed (0, 3, and... 
    nobody else reaches in-degree 0).
    result.size() (2) != n (6) → CYCLE!
```

---

## 🔹 Template Code — Kahn's Algorithm (BFS)

```cpp
#include <vector>
#include <queue>
using namespace std;

vector<int> topologicalSortKahn(vector<vector<int>>& adj, int n) {
    // Step 1: Calculate in-degree for all vertices
    vector<int> inDegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : adj[u])
            inDegree[v]++;
    
    // Step 2: Enqueue all vertices with in-degree 0
    queue<int> q;
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0)
            q.push(i);
    
    // Step 3: Process the queue
    vector<int> order;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);
        
        for (int v : adj[u]) {
            inDegree[v]--;
            if (inDegree[v] == 0)
                q.push(v);
        }
    }
    
    // Step 4: Cycle check
    // If order.size() != n, graph has a cycle!
    if ((int)order.size() != n) return {};
    return order;
}
```

**Usage:**

```cpp
int main() {
    int n = 6;
    vector<vector<int>> adj(n);
    adj[0] = {1, 3};  // CS101 -> CS201, MATH
    adj[1] = {2, 4};  // CS201 -> CS301, CS202
    adj[3] = {4};     // MATH  -> CS202
    adj[4] = {5};     // CS202 -> CS401

    vector<int> order = topologicalSortKahn(adj, n);

    if (order.empty()) {
        cout << "Cycle detected!\n";
    } else {
        for (int v : order) cout << v << " ";
        // Output: 0 1 3 2 4 5
    }
}
```

---

## 🔹 Template Code — DFS-Based (Reverse Post-Order)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

// DFS helper with cycle detection (3-state coloring)
bool dfs(int u, vector<vector<int>>& adj,
         vector<int>& state, vector<int>& order) {
    state[u] = 1;  // visiting (gray)
    
    for (int v : adj[u]) {
        if (state[v] == 1)                           // back edge!
            return false;                             // cycle detected
        if (state[v] == 0 && !dfs(v, adj, state, order))
            return false;                             // propagate cycle
    }
    
    state[u] = 2;          // done (black)
    order.push_back(u);    // post-order: push AFTER all descendants
    return true;
}

// DFS-based topological sort
// Returns empty vector if cycle exists
vector<int> topologicalSortDFS(vector<vector<int>>& adj, int n) {
    vector<int> state(n, 0);  // 0=unvisited, 1=visiting, 2=done
    vector<int> order;
    
    // Must try starting from EVERY unvisited node
    // (handles disconnected components)
    for (int i = 0; i < n; i++)
        if (state[i] == 0)
            if (!dfs(i, adj, state, order))
                return {};  // cycle exists, no valid ordering
    
    reverse(order.begin(), order.end());  // reverse post-order!
    return order;
}
```

---

## 🔹 When to Use Topological Sort

```
  +----------------------------------+----------------------------------+
  | Use Case                         | Example                          |
  +----------------------------------+----------------------------------+
  | Task scheduling                  | Project task ordering             |
  | Course prerequisites             | University degree planner         |
  | Build systems                    | Makefile, CMake, Gradle           |
  | Package managers                 | npm, pip, apt dependency          |
  |                                  |   resolution                     |
  | Spreadsheet cell evaluation      | Evaluate formulas in order        |
  | Compilation order                | Compile source files              |
  | Symbol resolution                | Linker dependency ordering        |
  | Data pipeline orchestration      | ETL job scheduling                |
  +----------------------------------+----------------------------------+
```

**Rule of thumb**: Anytime you have a set of tasks where some must happen before
others, and you need a valid execution order -- think topological sort.

---

## 🔹 Cycle Detection

Both approaches detect cycles, but differently:

```
  ╔═══════════════════════════════════════════════════════════════╗
  ║  Kahn's Algorithm                                           ║
  ║                                                             ║
  ║  If the result has fewer nodes than total → cycle exists.   ║
  ║  Nodes in a cycle never reach in-degree 0, so they're       ║
  ║  never enqueued and never processed.                        ║
  ║                                                             ║
  ║      A → B → C → A    in-degrees: A:1, B:1, C:1            ║
  ║      None have in-degree 0 → never processed → cycle!      ║
  ╠═══════════════════════════════════════════════════════════════╣
  ║  DFS-based                                                  ║
  ║                                                             ║
  ║  If you encounter a node in state GRAY → cycle!             ║
  ║  Gray = ancestor in current path. Reaching it again =       ║
  ║  path from it back to itself = cycle.                       ║
  ║                                                             ║
  ║      dfs(A): A=GRAY                                         ║
  ║        dfs(B): B=GRAY                                       ║
  ║          dfs(C): C=GRAY                                     ║
  ║            visit A → A is GRAY → CYCLE!                     ║
  ╚═══════════════════════════════════════════════════════════════╝
```

---

## 🔹 Kahn's vs DFS — Comparison

| Aspect | Kahn's (BFS) | DFS-based |
|---|---|---|
| **Style** | Iterative (queue) | Recursive (stack) |
| **Data structure** | Queue + in-degree array | Recursion stack + state array |
| **Cycle detection** | Check `result.size() != n` | Back-edge detection (gray → gray) |
| **Output order** | Direct topological order | Needs `reverse()` at the end |
| **Unique ordering** | Use `priority_queue` for lexicographically smallest | Harder to control order |
| **Best when** | Need to process in order, want layer-by-layer | Already doing DFS traversal |
| **Complexity** | O(V + E) time, O(V) space | O(V + E) time, O(V) space |
| **Intuition** | "Peel off ready nodes layer by layer" | "Go deep, finish descendants first" |

```
  Kahn's mental model:             DFS mental model:

  Layer 0:  [ A ]                  Start at A, go deep:
  Layer 1:  [ B, C ]               A → B → D → (done, push D)
  Layer 2:  [ D, E ]                         → (back to B, push B)
  Layer 3:  [ F ]                      → C → E → (done, push E)
                                             → (back to C, push C)
  Process layer by layer                → F → (done, push F)
  → A, B, C, D, E, F                → (back to A, push A)

                                   Stack: [D, B, E, C, F, A]
                                   Reverse: [A, F, C, E, B, D]
```

---

## 🔹 Common Pitfalls

**1. Forgetting it only works on DAGs**

```
  Topological sort is ONLY defined for Directed Acyclic Graphs.
    - Undirected graph? → Not applicable (A—B implies A→B AND B→A = cycle)
    - Has a cycle?      → Impossible. Detect and report it.

       A → B → C → A     ← No valid ordering possible
```

**2. Not handling disconnected components**

```
  Graph with disconnected parts:
       A → B       C → D       E (isolated)

  WRONG:  Start DFS only from node A
  RIGHT:  Start DFS from EVERY unvisited node

  If you only start from A, you'll never visit C, D, and E!

  for (int i = 0; i < n; i++)     ← This loop is critical
      if (state[i] == 0)
          dfs(i, ...);
```

**3. Confusing DFS order with topological order**

```
  Topological order is REVERSE POST-ORDER, not pre-order!

       A → C
       A → B → C

  Pre-order  (when you first visit):    A, B, C       or  A, C, B
  Post-order (when you finish):         C, B, A
  Topo order (reverse post-order):      A, B, C  ✓

  Pre-order can give:  A, C, B   ← WRONG! B should come before C
  Post-order reversed: A, B, C   ← ALWAYS CORRECT
```

**4. Forgetting to reverse the DFS result**

```
  DFS pushes nodes when they FINISH (post-order).
  The last node to finish is the one with no prerequisites →
  it should come FIRST.

  Without reverse: [2, 5, 4, 1, 3, 0]  ← WRONG (backwards)
  With reverse:    [0, 3, 1, 4, 5, 2]  ← CORRECT

  Don't forget:  reverse(order.begin(), order.end());
```

**5. Using topological sort on undirected graphs**

```
  Topological sort is only defined for DIRECTED graphs.
  An undirected edge A—B implies both A→B and B→A,
  which is a cycle of length 2. Always impossible.
```

---

## 🔹 Complexity

```
  +-------------------+-------------------+
  | Time Complexity   |  O(V + E)         |
  +-------------------+-------------------+
  | Space Complexity  |  O(V + E)         |
  +-------------------+-------------------+

  Both Kahn's and DFS have the same complexity.
  V = number of vertices, E = number of edges.
  We visit every vertex once and traverse every edge once.
```

---

**See also:** [[BFS - Breadth First Search]], [[DFS - Depth First Search]], [[Graph Representations]]
