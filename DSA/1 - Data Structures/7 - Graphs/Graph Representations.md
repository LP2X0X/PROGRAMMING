---
tags:
  - algorithms
  - data-structure
  - graph
---

## 🔹 Real-World Analogy

Think of a city road map. Each intersection is a **vertex** and each road between intersections is an **edge**. Some roads are one-way (directed), some have speed limits or distances (weighted), and some neighborhoods have many connecting roads (dense) while rural areas have few (sparse).

- **Social networks**: people are vertices, friendships are edges (undirected), "follows" are directed edges.
- **The internet**: web pages are vertices, hyperlinks are directed edges.
- **Flight routes**: airports are vertices, flights are weighted directed edges (weight = distance, cost, or time).

How you choose to store this structure in memory determines how fast you can answer questions like "is there a road from A to B?" or "which cities can I reach from here?"

---

## 🔹 The Example Graph

Every representation below uses the **same** directed, weighted graph with 5 nodes:

```
         2
  (0)-------->(1)
   |         / |
 4 |       /   | 3
   |    7/     |
   v  /        v
  (2)<--------(3)
   |      1     
 5 |             
   v             
  (4)            
```

**Edge list (directed, weighted):**

| From | To | Weight |
|------|----|--------|
| 0    | 1  | 2      |
| 0    | 2  | 4      |
| 1    | 2  | 7      |
| 1    | 3  | 3      |
| 3    | 2  | 1      |
| 2    | 4  | 5      |

Vertices: {0, 1, 2, 3, 4} -- 5 vertices, 6 edges.

---

## 🔹 Representation 1: Adjacency Matrix

A 2D array `M[V][V]` where `M[u][v]` stores the edge weight from `u` to `v`. Use a sentinel value (e.g., 0 or INF) to indicate "no edge."

### How It Looks

```
     To:   0     1     2     3     4
         +-----+-----+-----+-----+-----+
From 0:  |  0  |  2  | 4   |  .  |  .  |
         +-----+-----+-----+-----+-----+
     1:  |  .  |  0  |  7  |  3  |  .  |
         +-----+-----+-----+-----+-----+
     2:  |  .  |  .  |  0  |  .  |  5  |
         +-----+-----+-----+-----+-----+
     3:  |  .  |  .  |  1  |  0  |  .  |
         +-----+-----+-----+-----+-----+
     4:  |  .  |  .  |  .  |  .  |  0  |
         +-----+-----+-----+-----+-----+

( . = INF / no edge )
```

### Pros

- O(1) edge lookup: just check `M[u][v]`.
- Simple to implement and reason about.
- Easy to detect and work with **dense** graphs.
- Matrix operations (e.g., transitive closure via matrix multiplication) are natural.

### Cons

- O(V^2) space regardless of how many edges exist -- wasteful for sparse graphs.
- Iterating over all neighbors of a vertex is O(V) even if it only has a few.
- Adding a vertex requires resizing the entire matrix.

### When to Use

- Dense graphs where E is close to V^2.
- When you need O(1) edge existence checks.
- Floyd-Warshall (all-pairs shortest paths) operates directly on the matrix.
- Small graphs where V^2 memory is not a concern.

---

## 🔹 Representation 2: Adjacency List

An array of V lists (vectors). `adj[u]` contains all pairs `(v, weight)` for edges leaving `u`.

### How It Looks

```
adj[0] --> [(1, 2), (2, 4)]
adj[1] --> [(2, 7), (3, 3)]
adj[2] --> [(4, 5)]
adj[3] --> [(2, 1)]
adj[4] --> []
```

Visualized as a structure:

```
  Index    Linked neighbors
  +---+
  | 0 | --> (1,w=2) --> (2,w=4) --> null
  +---+
  | 1 | --> (2,w=7) --> (3,w=3) --> null
  +---+
  | 2 | --> (4,w=5) --> null
  +---+
  | 3 | --> (2,w=1) --> null
  +---+
  | 4 | --> null
  +---+
```

### Pros

- Space efficient: O(V + E). Only stores edges that exist.
- Iterating over neighbors of a vertex is O(degree(v)) -- fast for sparse graphs.
- Adding an edge is O(1) (push to list).
- The default choice for most graph algorithms (BFS, DFS, Dijkstra, topological sort).

### Cons

- Edge existence check is O(degree(v)) -- must scan the neighbor list.
- Slightly more complex than a matrix to implement.
- Cache performance can suffer with pointer-based linked lists (prefer `vector` in C++).

### When to Use

- Sparse graphs (E much less than V^2) -- the vast majority of real-world graphs.
- BFS, DFS, Dijkstra, Bellman-Ford, topological sort, Tarjan's SCC.
- When you need to iterate over neighbors frequently.

---

## 🔹 Representation 3: Edge List

A flat list of all edges, each stored as `(u, v, weight)`.

### How It Looks

```
edges[] = {
    (0, 1, 2),
    (0, 2, 4),
    (1, 2, 7),
    (1, 3, 3),
    (3, 2, 1),
    (2, 4, 5)
}
```

```
  +----------+----------+----------+
  | (0,1, 2) | (0,2, 4) | (1,2, 7) |
  +----------+----------+----------+
  | (1,3, 3) | (3,2, 1) | (2,4, 5) |
  +----------+----------+----------+
```

### Pros

- Simplest representation: just a list of tuples.
- Very efficient for **edge-centric** algorithms (Kruskal's MST sorts edges by weight).
- Easy to read from input and pass around.
- O(E) space -- minimal overhead.

### Cons

- Checking if a specific edge exists requires O(E) scan (or O(log E) with sorting + binary search).
- Finding all neighbors of a vertex requires O(E) scan.
- Not suitable for algorithms that need fast neighbor iteration (BFS, DFS, Dijkstra).

### When to Use

- Kruskal's MST (sort edges, union-find).
- Bellman-Ford (iterates over all edges V-1 times).
- When input is given as edge pairs and you do not need neighbor lookups.
- As an intermediate form before building an adjacency list.

---

## 🔹 Side-by-Side Comparison

| Property            | Adjacency Matrix        | Adjacency List         | Edge List              |
|---------------------|-------------------------|------------------------|------------------------|
| **Space**           | O(V^2)                  | O(V + E)               | O(E)                   |
| **Edge lookup**     | O(1)                    | O(degree(v))           | O(E)                   |
| **Add edge**        | O(1)                    | O(1) amortized         | O(1) amortized         |
| **Remove edge**     | O(1)                    | O(degree(v))           | O(E)                   |
| **Iterate neighbors** | O(V)                  | O(degree(v))           | O(E)                   |
| **Iterate all edges** | O(V^2)                | O(V + E)               | O(E)                   |
| **Best for**        | Dense, Floyd-Warshall   | BFS, DFS, Dijkstra     | Kruskal's, Bellman-Ford|
| **Worst for**       | Sparse, large V         | O(1) edge checks       | Neighbor iteration     |

---

## 🔹 Template Code (C++)

### Building Adjacency List from Edge Input

```cpp
#include <vector>
using namespace std;

// Weighted directed graph
struct Edge {
    int to, weight;
};

// V = number of vertices, edges given as (from, to, weight)
vector<vector<Edge>> buildAdjList(int V, 
        const vector<tuple<int,int,int>>& edges, 
        bool directed = true) {
    vector<vector<Edge>> adj(V);
    for (auto& [u, v, w] : edges) {
        adj[u].push_back({v, w});
        if (!directed) {
            adj[v].push_back({u, w});  // add reverse edge
        }
    }
    return adj;
}
```

### Building Adjacency Matrix

```cpp
#include <vector>
using namespace std;

const int INF = 1e9;

vector<vector<int>> buildAdjMatrix(int V,
        const vector<tuple<int,int,int>>& edges,
        bool directed = true) {
    // Initialize with INF (no edge), 0 on diagonal
    vector<vector<int>> mat(V, vector<int>(V, INF));
    for (int i = 0; i < V; i++) mat[i][i] = 0;

    for (auto& [u, v, w] : edges) {
        mat[u][v] = w;
        if (!directed) {
            mat[v][u] = w;
        }
    }
    return mat;
}
```

### Converting Adjacency List to Matrix

```cpp
vector<vector<int>> adjListToMatrix(int V, 
        const vector<vector<Edge>>& adj) {
    vector<vector<int>> mat(V, vector<int>(V, INF));
    for (int i = 0; i < V; i++) mat[i][i] = 0;

    for (int u = 0; u < V; u++) {
        for (auto& [v, w] : adj[u]) {
            mat[u][v] = w;
        }
    }
    return mat;
}
```

### Converting Matrix to Adjacency List

```cpp
vector<vector<Edge>> matrixToAdjList(int V, 
        const vector<vector<int>>& mat) {
    vector<vector<Edge>> adj(V);
    for (int u = 0; u < V; u++) {
        for (int v = 0; v < V; v++) {
            if (u != v && mat[u][v] != INF) {
                adj[u].push_back({v, mat[u][v]});
            }
        }
    }
    return adj;
}
```

### Converting Edge List to Adjacency List (Most Common Input Pattern)

This is the pattern you will use most often in competitive programming and interview problems:

```cpp
// Typical input: first line = V E, then E lines of (u, v, w)
int V, E;
cin >> V >> E;

vector<vector<Edge>> adj(V);
for (int i = 0; i < E; i++) {
    int u, v, w;
    cin >> u >> v >> w;
    adj[u].push_back({v, w});
    // adj[v].push_back({u, w});  // uncomment for undirected
}
```

---

## 🔹 Common Pitfalls

### 1. Forgetting Both Directions for Undirected Graphs

The single most common bug. If the graph is undirected, you must add the edge in **both** directions:

```cpp
// WRONG -- only adds one direction
adj[u].push_back({v, w});

// CORRECT -- add both
adj[u].push_back({v, w});
adj[v].push_back({u, w});
```

For adjacency matrices, set both `mat[u][v]` and `mat[v][u]`.

### 2. 0-Indexed vs 1-Indexed Nodes

Many problems use 1-indexed vertices in the input. If you use a 0-indexed adjacency list of size V, reading 1-indexed input without converting causes out-of-bounds access or missing node 0.

```cpp
// Option A: convert to 0-indexed
adj[u - 1].push_back({v - 1, w});

// Option B: allocate V+1 entries and ignore index 0
vector<vector<Edge>> adj(V + 1);
```

### 3. Confusing Directed and Undirected

Read the problem statement carefully. "Road between A and B" usually means **undirected**. "Flight from A to B" usually means **directed**.

### 4. Self-Loops and Parallel Edges

- Adjacency matrices naturally overwrite parallel edges (only the last weight survives).
- Adjacency lists can store parallel edges (multiple entries for same destination).
- Know whether the problem allows self-loops (`u == v`) and handle accordingly.

### 5. Using Adjacency Matrix for Large V

If V = 100,000, the matrix requires 10^10 entries -- instant memory limit exceeded. Use an adjacency list.

Rule of thumb: if V > 10,000, use adjacency list. If V <= 1,000, matrix is fine.

---

## 🔹 Decision Flowchart

```
Is V small (< 1000)?
├── YES --> Is the graph dense (E ~ V^2)?
│           ├── YES --> Adjacency Matrix
│           └── NO  --> Adjacency List
└── NO  --> Adjacency List (matrix won't fit in memory)

Need edge-centric operations (sort by weight, iterate all edges)?
├── YES --> Edge List (possibly alongside adjacency list)
└── NO  --> Adjacency List

Need O(1) edge lookup?
├── YES and V is small --> Adjacency Matrix
├── YES and V is large --> Adjacency List + unordered_set for lookup
└── NO  --> Adjacency List
```

---

## 🔹 Related Notes

- [[Graph Terminology]] -- definitions of vertex, edge, directed, weighted, degree, cycle, etc.
