---
tags:
  - algorithms
  - graph
  - dfs
---

# DFS - Depth First Search

## 🔹 Real-World Analogy

**Exploring a maze.** You walk forward as far as you can down one corridor. When you hit a dead end, you backtrack to the last intersection and try a different corridor. You keep going deep before trying alternatives.

Like reading a "choose your own adventure" book by always following the first option until you reach an ending, then going back to try the other choices.

```
Maze exploration (DFS style):

    START
      |
      v
    +---+---+---+---+
    |   | 1 → 2 → 3 |     DFS goes: 1 → 2 → 3 (dead end!)
    +   +   +---+   +                backtrack to 2
    |   | ↓     |   |                2 → 4 (dead end!)
    +   +---+   +   +                backtrack to 1
    |   | 4   |   |   |              1 → 5 → 6 → EXIT!
    +   +   +   +   +
    | 5 → → → 6 → EXIT|
    +---+---+---+---+
```

## 🔹 What is DFS?

DFS explores as **deep as possible** along each branch before backtracking. It uses a [[Stack]] (or the recursion call stack — which IS a stack).

```
DFS = Stack (or Recursion) + Visited Set + Go Deep First
```

**Key difference from BFS:** BFS goes wide (level by level), DFS goes deep (path by path).

```
         1                  BFS order: 1, 2, 3, 4, 5, 6, 7
        / \                 DFS order: 1, 2, 4, 5, 3, 6, 7
       2   3
      / \ / \
     4  5 6  7
```

## 🔹 Step-by-Step ASCII Walkthrough

**Same sample graph as BFS:**

```
    1 --- 2
    |     |
    3 --- 4 --- 5
```

**Adjacency list:**

```
1: [2, 3]
2: [1, 4]
3: [1, 4]
4: [2, 3, 5]
5: [4]
```

**DFS starting from node 1 (recursive, visiting neighbors in list order):**

```
Call Stack          Action                      Visited         Order
────────────────────────────────────────────────────────────────────────
dfs(1)              Visit 1                     {1}             [1]
  dfs(2)            Visit 2 (1st neighbor of 1) {1,2}           [1,2]
    skip 1          1 already visited
    dfs(4)          Visit 4 (2nd neighbor of 2) {1,2,4}         [1,2,4]
      skip 2        2 already visited
      dfs(3)        Visit 3 (2nd neighbor of 4) {1,2,4,3}       [1,2,4,3]
        skip 1      1 already visited
        skip 4      4 already visited
      return from 3
      dfs(5)        Visit 5 (3rd neighbor of 4) {1,2,4,3,5}     [1,2,4,3,5]
        skip 4      4 already visited
      return from 5
    return from 4
  return from 2
  skip 3            3 already visited
return from 1       DONE!
```

**DFS Tree:**

```
    1
    |
    2
    |
    4
   / \
  3   5
```

**Traversal order: 1 → 2 → 4 → 3 → 5**

Notice DFS went deep (1→2→4) before backtracking to explore 3 and 5. Compare with BFS order: 1 → 2, 3 → 4 → 5 (level by level).

## 🔹 Recursive vs Iterative Implementation

**Recursive DFS** is the most natural and commonly used form. The function call stack acts as the implicit stack.

```
Recursive DFS call stack visualization:

    dfs(1) ──┐
       dfs(2) ──┐
          dfs(4) ──┐
             dfs(3) ──┐
             return ◄──┘
             dfs(5) ──┐
             return ◄──┘
          return ◄──┘
       return ◄──┘
    return ◄──┘

    Each level = one frame on the call stack
    Backtracking = returning from a function call
```

**Iterative DFS** uses an explicit [[Stack]]. Useful when recursion depth might cause stack overflow.

```
Iterative DFS with explicit stack:

    Stack: [1]
    Pop 1, push neighbors [3, 2]     → Stack: [3, 2]
    Pop 2, push neighbors [4]        → Stack: [3, 4]    (1 visited)
    Pop 4, push neighbors [5, 3, 2]  → Stack: [3, 5, 3] (2 visited)
    Pop 3, push neighbors []         → Stack: [3, 5]    (1,4 visited)
    Pop 5, push neighbors []         → Stack: [3]       (4 visited)
    Pop 3, already visited, skip     → Stack: []
    DONE!

    Note: iterative DFS may visit nodes in a different order
    than recursive DFS due to stack LIFO behavior.
```

## 🔹 DFS on Graphs vs Trees

```
TREE (no cycles):              GRAPH (has cycles):
    1                              1 --- 2
   / \                             |     |
  2   3     No visited set         3 --- 4
 / \        needed! No cycles.
4   5                          MUST use visited set
                               or you loop forever!
```

| Aspect | Tree | Graph |
|--------|------|-------|
| Cycles? | No | Possible |
| Visited set? | Not needed | **Required** |
| Parent tracking? | Optional | Important for back-edge detection |

## 🔹 Complexity

| Metric | Complexity | Why |
|--------|-----------|-----|
| **Time** | **O(V + E)** | Each vertex visited once. Each edge examined once (twice for undirected). |
| **Space** | **O(V)** | Visited set: O(V). Recursion stack depth: O(V) worst case. |

See [[Big O - Definition]].

**Space comparison with BFS:**

```
Narrow, deep graph:          Wide, shallow graph:
    1                            1
    |                         / | | | \
    2                        2 3 4 5 6 7
    |
    3       DFS: O(depth)        DFS: O(1) stack
    |       = O(V) here         BFS: O(width) = O(V)
    4
    |       BFS: O(1) queue
    5       (only 1 node per level)
```

DFS is more space-efficient on wide graphs. BFS is more space-efficient on deep, narrow graphs. In the worst case, both are O(V).

## 🔹 Pre-order vs Post-order in DFS

This distinction is crucial for many algorithms, especially [[Topological Sort]].

```
         A
        / \
       B   C
      / \
     D   E

Pre-order  (process BEFORE children):  A, B, D, E, C
    → "entering" the node
    → useful for: copying a tree, prefix expressions

Post-order (process AFTER children):   D, E, B, C, A
    → "leaving" the node
    → useful for: topological sort, tree deletion, postfix expressions
```

```cpp
void dfs(int node) {
    visited[node] = true;
    
    // ← PRE-ORDER processing here (before exploring children)
    //   e.g., print node, record discovery time
    
    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfs(neighbor);
        }
    }
    
    // ← POST-ORDER processing here (after all children explored)
    //   e.g., push to stack for topological sort, record finish time
}
```

## 🔹 Applications

| Application | How DFS is used |
|-------------|-----------------|
| **Cycle detection** | If DFS encounters a node already in the current recursion path (back edge), there's a cycle |
| **[[Topological Sort]]** | Reverse post-order of DFS gives topological ordering of a DAG |
| **Connected components** | Run DFS from each unvisited node; each DFS run = one component |
| **Path finding** | DFS finds *a* path (not necessarily shortest) between two nodes |
| **Maze solving** | Go deep, backtrack at dead ends — DFS naturally solves mazes |
| **Island counting (grids)** | Flood fill with DFS: when you find land, DFS to mark entire island |
| **Bridges & articulation points** | Track discovery/low times during DFS (Tarjan's algorithm) |

## 🔹 Template Code — Recursive DFS (Graph)

```cpp
// Recursive DFS on adjacency list
void dfs(vector<vector<int>>& adj, int node, vector<bool>& visited) {
    visited[node] = true;
    // Process node (pre-order)
    
    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfs(adj, neighbor, visited);
        }
    }
    // Process node (post-order)
}

// Usage: find all connected components
int countComponents(vector<vector<int>>& adj, int n) {
    vector<bool> visited(n, false);
    int components = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(adj, i, visited);
            components++;
        }
    }
    return components;
}
```

## 🔹 Template Code — Iterative DFS (Graph)

```cpp
// Iterative DFS using explicit stack
void dfsIterative(vector<vector<int>>& adj, int start) {
    int n = adj.size();
    vector<bool> visited(n, false);
    stack<int> stk;
    
    stk.push(start);
    
    while (!stk.empty()) {
        int node = stk.top();
        stk.pop();
        
        if (visited[node]) continue;
        visited[node] = true;
        // Process node
        
        for (int neighbor : adj[node]) {
            if (!visited[neighbor]) {
                stk.push(neighbor);
            }
        }
    }
}
```

**Note:** Iterative DFS checks `visited` after popping (not before pushing) because the same node might be pushed multiple times from different neighbors. This is a key difference from BFS where we mark visited when enqueueing.

## 🔹 Template Code — Grid DFS (Island Counting)

```cpp
// DFS flood fill on a grid
void dfsGrid(vector<vector<int>>& grid, int r, int c) {
    int rows = grid.size(), cols = grid[0].size();
    if (r < 0 || r >= rows || c < 0 || c >= cols) return;
    if (grid[r][c] != 1) return;  // not land or already visited
    
    grid[r][c] = 0;  // mark visited by sinking the island
    
    dfsGrid(grid, r+1, c);  // down
    dfsGrid(grid, r-1, c);  // up
    dfsGrid(grid, r, c+1);  // right
    dfsGrid(grid, r, c-1);  // left
}

int countIslands(vector<vector<int>>& grid) {
    int count = 0;
    for (int r = 0; r < grid.size(); r++) {
        for (int c = 0; c < grid[0].size(); c++) {
            if (grid[r][c] == 1) {
                dfsGrid(grid, r, c);
                count++;
            }
        }
    }
    return count;
}
```

```
Island counting example:

Grid:               After DFS from (0,0):    After DFS from (2,2):
1 1 0 0 0           0 0 0 0 0                0 0 0 0 0
1 1 0 0 0     →     0 0 0 0 0          →     0 0 0 0 0
0 0 1 0 0           0 0 1 0 0                0 0 0 0 0
0 0 0 1 1           0 0 0 1 1                0 0 0 1 1

count = 1            count = 1                count = 2
                                              (one more DFS at (3,3) → count = 3)
```

## 🔹 BFS vs DFS Comparison

| Aspect | BFS | DFS |
|--------|-----|-----|
| **Data structure** | [[Queue]] (FIFO) | [[Stack]] / [[Recursion]] (LIFO) |
| **Exploration order** | Level by level (wide) | Path by path (deep) |
| **Shortest path (unweighted)** | **YES** | No |
| **Space (worst case)** | O(width of graph) | O(depth of graph) |
| **Complete?** | Yes (finds all reachable) | Yes (finds all reachable) |
| **Best for** | Shortest path, level-order, nearest target | Cycle detection, topological sort, exhaustive search |
| **Implementation** | Always iterative (queue) | Usually recursive, sometimes iterative |

```
When to use which:

Need shortest path?          → BFS
Need to explore all paths?   → DFS
Cycle detection?             → DFS (back edges)
Topological sort?            → DFS (post-order)
Level-order traversal?       → BFS
Connected components?        → Either works
Memory constrained + deep?   → BFS (narrower frontier)
Memory constrained + wide?   → DFS (smaller stack)
```

## 🔹 Common Pitfalls

**1. Forgetting visited set on graphs**

```
Works on trees:         Breaks on graphs:
    1                       1 --- 2
   / \                      |     |
  2   3   (no cycles)       3 --- 4   (cycles → infinite loop!)

Trees: no visited set needed.
Graphs: ALWAYS use a visited set.
```

**2. Stack overflow on deep graphs**

```
If your graph has V = 100,000 nodes in a chain:
    1 → 2 → 3 → ... → 100000

Recursive DFS creates 100,000 stack frames → STACK OVERFLOW!
Solution: use iterative DFS with explicit stack.
```

**3. Confusing pre-order and post-order processing**

```
For topological sort, you need POST-order (then reverse).
Using PRE-order gives the wrong result!

Pre-order:  process when ENTERING the node  (discovery)
Post-order: process when LEAVING the node   (finish)
```

---

**See also:** [[Stack]], [[Graph Representations]], [[BFS - Breadth First Search]], [[Recursion]], [[Backtracking]], [[Topological Sort]]
