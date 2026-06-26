---
tags:
  - algorithms
  - graph
  - bfs
---

# BFS - Breadth First Search

## 🔹 Real-World Analogy

**Ripples in a pond.** Drop a stone into still water and waves expand outward in concentric circles. The closest points are reached first, then the next ring, then the next. BFS works the same way — it explores all nodes at distance 1, then distance 2, then distance 3, and so on.

Another way to think about it: **searching for someone at a party.** You ask all your direct friends first (distance 1). If none of them have seen the person, you ask all of *their* friends (distance 2), and so on. You never jump ahead to ask a stranger before exhausting your closer connections.

```
Drop stone at S:

  Ring 0:     S           ← start here
  Ring 1:   A B C         ← distance 1 (direct neighbors)
  Ring 2:  D E F G        ← distance 2
  Ring 3: H I J K L       ← distance 3
```

## 🔹 What is BFS?

BFS explores a graph **level by level** using a [[Queue]] (FIFO — First In, First Out). It visits all neighbors of the current node before moving to the next level of depth.

**Core idea:** Use a queue to process nodes in the order they were discovered. The first time you reach a node is always via the shortest path (in an unweighted graph).

```
BFS = Queue + Visited Set + Level-by-Level Exploration
```

## 🔹 Step-by-Step ASCII Walkthrough

**Sample graph:**

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

**BFS starting from node 1:**

```
Step 1: Visit 1, mark visited, enqueue neighbors
         Visited: {1}
         Queue: [2, 3]          ← neighbors of 1
         Order: [1]

Step 2: Dequeue 2, visit it, enqueue unvisited neighbors
         Visited: {1, 2}
         Queue: [3, 4]          ← 1 already visited, 4 is new
         Order: [1, 2]

Step 3: Dequeue 3, visit it, enqueue unvisited neighbors
         Visited: {1, 2, 3}
         Queue: [4]             ← 1 already visited, 4 already in queue
         Order: [1, 2, 3]

Step 4: Dequeue 4, visit it, enqueue unvisited neighbors
         Visited: {1, 2, 3, 4}
         Queue: [5]             ← 2,3 already visited, 5 is new
         Order: [1, 2, 3, 4]

Step 5: Dequeue 5, visit it, no unvisited neighbors
         Visited: {1, 2, 3, 4, 5}
         Queue: []              ← empty, BFS complete
         Order: [1, 2, 3, 4, 5]
```

**BFS Tree (showing discovery edges):**

```
        1           Level 0
       / \
      2   3         Level 1
      |
      4             Level 2
      |
      5             Level 3
```

**Distance from source (node 1):**

```
Node:     1  2  3  4  5
Distance: 0  1  1  2  3
```

## 🔹 The Visited Set — Why It's Essential

In a graph with cycles, without a visited set, BFS would loop forever:

```
Without visited set on a cycle:
    1 → 2 → 4 → 3 → 1 → 2 → 4 → 3 → 1 → ...  (infinite!)

    1 --- 2
    |     |       ← this forms a cycle: 1→2→4→3→1
    3 --- 4
```

**Rule:** Mark a node as visited **when you enqueue it**, not when you dequeue it. This prevents the same node from being added to the queue multiple times.

```
WRONG:  enqueue first, mark visited when dequeuing
        → node can be enqueued multiple times → wastes memory + time

RIGHT:  mark visited immediately when enqueueing
        → each node enters the queue exactly once
```

## 🔹 BFS Guarantees Shortest Path in Unweighted Graphs

**Why?** Because BFS explores nodes in strict order of their distance from the source. All nodes at distance *d* are processed before any node at distance *d+1*.

```
Source S:
  d=0: S processed first
  d=1: all neighbors of S processed next
  d=2: all nodes 2 edges away processed next
  ...

The FIRST time BFS reaches a node = shortest path to that node.
No shorter path can exist because all shorter paths were already explored.
```

**This does NOT work for weighted graphs.** In a weighted graph, a path with more edges can have a smaller total weight. For weighted shortest paths, use [[Dijkstra Algorithm]].

```
Weighted counterexample:
    A --1-- B --1-- C       Path A→B→C = weight 2
    |                       Path A→C   = weight 10
    +------10------→C

    BFS finds A→C (1 edge) before A→B→C (2 edges)
    But A→B→C is actually shorter by weight!
```

## 🔹 Complexity

| Metric | Complexity | Why |
|--------|-----------|-----|
| **Time** | **O(V + E)** | Every vertex is enqueued/dequeued once (O(V)). Every edge is examined once when processing its endpoints (O(E)). |
| **Space** | **O(V)** | Queue can hold up to O(V) nodes. Visited set stores V entries. |

Where V = number of vertices, E = number of edges. See [[Big O - Definition]].

## 🔹 Applications

| Application | Description |
|-------------|-------------|
| **Shortest path (unweighted)** | Find minimum number of edges between two nodes |
| **Level-order tree traversal** | Visit tree nodes level by level (root, then children, then grandchildren) |
| **Connected components** | Find all nodes reachable from a starting node; repeat for disconnected parts |
| **Word ladder** | Transform "hit" → "cog" changing one letter at a time (each word is a node) |
| **Nearest target** | Find the closest target in a grid (e.g., nearest exit, nearest hospital) |
| **Bipartiteness check** | 2-color the graph level by level — if a conflict arises, it's not bipartite |

## 🔹 BFS on a Grid

Treat each cell as a node. Neighbors are the 4 (or 8) adjacent cells. This is extremely common in coding problems.

```
Grid (4x4):              Coordinates:
+---+---+---+---+        (0,0) (0,1) (0,2) (0,3)
| . | . | . | . |        (1,0) (1,1) (1,2) (1,3)
+---+---+---+---+        (2,0) (2,1) (2,2) (2,3)
| . | S | . | . |        (3,0) (3,1) (3,2) (3,3)
+---+---+---+---+
| . | . | . | E |        S = Start (1,1)
+---+---+---+---+        E = End   (2,3)
| . | . | . | . |
+---+---+---+---+

4-directional neighbors of (1,1):
              (0,1)           ← up
       (1,0)  (1,1)  (1,2)   ← left, center, right
              (2,1)           ← down
```

**Direction vectors** (the key trick for grid traversal):

```cpp
int dx[] = {0, 0, 1, -1};   // row changes:    right, left, down, up
int dy[] = {1, -1, 0, 0};   // column changes:  right, left, down, up

// To get all 4 neighbors of (r, c):
for (int i = 0; i < 4; i++) {
    int nr = r + dx[i];
    int nc = c + dy[i];
    // check bounds, then process (nr, nc)
}
```

## 🔹 Template Code — Graph BFS

```cpp
// BFS on adjacency list graph
// Returns: shortest distance from src to all nodes (-1 if unreachable)
vector<int> bfs(vector<vector<int>>& adj, int src) {
    int n = adj.size();
    vector<int> dist(n, -1);
    queue<int> q;
    
    dist[src] = 0;
    q.push(src);
    
    while (!q.empty()) {
        int node = q.front();
        q.pop();
        
        for (int neighbor : adj[node]) {
            if (dist[neighbor] == -1) {  // not visited
                dist[neighbor] = dist[node] + 1;
                q.push(neighbor);
            }
        }
    }
    return dist;
}
```

**Usage:**

```cpp
// Build graph with 6 nodes (0-5)
vector<vector<int>> adj(6);
adj[0].push_back(1); adj[1].push_back(0);  // edge 0-1
adj[0].push_back(2); adj[2].push_back(0);  // edge 0-2
adj[1].push_back(3); adj[3].push_back(1);  // edge 1-3
// ...

vector<int> distances = bfs(adj, 0);
// distances[i] = shortest distance from node 0 to node i
```

## 🔹 Template Code — Grid BFS

```cpp
// BFS on a grid — find shortest path from (sr,sc) to (er,ec)
// grid[r][c] = 0 means passable, 1 means wall
int gridBFS(vector<vector<int>>& grid, int sr, int sc, int er, int ec) {
    int rows = grid.size(), cols = grid[0].size();
    if (grid[sr][sc] == 1 || grid[er][ec] == 1) return -1;
    
    int dx[] = {0, 0, 1, -1};
    int dy[] = {1, -1, 0, 0};
    
    vector<vector<bool>> visited(rows, vector<bool>(cols, false));
    queue<tuple<int,int,int>> q;  // {row, col, distance}
    
    q.push({sr, sc, 0});
    visited[sr][sc] = true;
    
    while (!q.empty()) {
        auto [r, c, dist] = q.front();
        q.pop();
        
        if (r == er && c == ec) return dist;
        
        for (int i = 0; i < 4; i++) {
            int nr = r + dx[i], nc = c + dy[i];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols 
                && !visited[nr][nc] && grid[nr][nc] == 0) {
                visited[nr][nc] = true;
                q.push({nr, nc, dist + 1});
            }
        }
    }
    return -1;  // unreachable
}
```

## 🔹 Common Pitfalls

**1. Marking visited AFTER dequeuing instead of BEFORE enqueueing**

```
WRONG — node gets enqueued multiple times:
    queue: [A]
    dequeue A → enqueue B, C     queue: [B, C]
    dequeue B → enqueue C (!)    queue: [C, C]  ← duplicate!

RIGHT — mark visited when enqueueing:
    queue: [A]
    dequeue A → mark B,C visited, enqueue B,C   queue: [B, C]
    dequeue B → C already visited, skip          queue: [C]
```

**2. Using BFS for weighted shortest path**

BFS counts edges, not weights. For weighted graphs, use [[Dijkstra Algorithm]].

**3. Off-by-one in grid bounds checking**

```cpp
// WRONG: using <= instead of <
if (nr >= 0 && nr <= rows && nc >= 0 && nc <= cols)  // out of bounds!

// RIGHT:
if (nr >= 0 && nr < rows && nc >= 0 && nc < cols)
```

---

**See also:** [[Queue]], [[Graph Representations]], [[DFS - Depth First Search]], [[Dijkstra Algorithm]]
