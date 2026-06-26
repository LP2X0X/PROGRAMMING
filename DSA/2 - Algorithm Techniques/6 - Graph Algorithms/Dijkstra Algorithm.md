---
tags:
  - algorithms
  - graph
  - shortest-path
---

# Dijkstra Algorithm

## 🔹 Real-World Analogy

**GPS navigation.** You want to find the fastest route from home to work. Roads have different speeds and distances (weights). Your GPS always expands the closest unvisited location first — it figures out the nearest reachable point, locks it in, then considers where to go next from there.

```
            5 min
    Home ─────────→ Gas Station
      |                  |
      | 2 min            | 1 min
      |                  |
      v                  v
    Park ────────→ Office
          3 min

    GPS tries shortest first:
    Home (0) → Park (2) → Gas Station (5) vs Park→Office (5) → Office (5)
    Shortest to Office = 5 min via Home→Park→Office
```

## 🔹 What is Dijkstra's Algorithm?

Dijkstra's finds the **shortest path from a single source to ALL vertices** in a **weighted graph with non-negative edge weights**. It uses a [[Priority Queue]] (min-heap) to always process the closest unvisited node next.

```
Dijkstra = BFS + Edge Weights + Priority Queue
         = "Greedy" shortest path (always pick the minimum)
```

**Key constraint:** All edge weights must be **non-negative** (>= 0). If you have negative weights, use Bellman-Ford instead.

## 🔹 The Relaxation Concept

"Relaxing" an edge means: **checking if going through a node gives a shorter path.**

```
Current knowledge: dist[v] = 10  (best known path to v)
New candidate:     dist[u] + weight(u,v) = 7

Since 7 < 10, we "relax" the edge:
    dist[v] = 7  (found a shorter path!)
    parent[v] = u (remember how we got there)
```

```cpp
// Relaxation — the core operation
if (dist[u] + weight(u, v) < dist[v]) {
    dist[v] = dist[u] + weight(u, v);  // "relax" edge u→v
}
```

```
Visual: relaxation of edge B→D

Before:                          After:
A ──2──→ B                       A ──2──→ B
         |                                |
         1 (relax!)                       1
         |                                |
         v                                v
         D  dist[D] = ∞                   D  dist[D] = 3
                                             (dist[B] + weight(B,D) = 2 + 1)
```

## 🔹 Step-by-Step Walkthrough

**Weighted directed graph:**

```
        2
    A ────→ B
    |       |  \
   4|      1|   3
    |       |    \
    v       v     v
    C ────→ D ───→ E
        3       2
```

**Edge list:** A→B(2), A→C(4), B→D(1), B→E(3), C→D(3), D→E(2)

**Dijkstra from source A:**

```
INITIALIZATION:
    dist  = {A:0, B:∞, C:∞, D:∞, E:∞}
    PQ    = [(0, A)]

STEP 1: Process A (dist=0)
    Relax A→B: dist[B] = min(∞, 0+2) = 2  ✓ updated
    Relax A→C: dist[C] = min(∞, 0+4) = 4  ✓ updated

    dist  = {A:0, B:2, C:4, D:∞, E:∞}
    PQ    = [(2, B), (4, C)]

    Settled: {A}

STEP 2: Process B (dist=2)  ← smallest in PQ
    Relax B→D: dist[D] = min(∞, 2+1) = 3  ✓ updated
    Relax B→E: dist[E] = min(∞, 2+3) = 5  ✓ updated

    dist  = {A:0, B:2, C:4, D:3, E:5}
    PQ    = [(3, D), (4, C), (5, E)]

    Settled: {A, B}

STEP 3: Process D (dist=3)  ← smallest in PQ
    Relax D→E: dist[E] = min(5, 3+2) = 5  ✗ no improvement (5 = 5)

    dist  = {A:0, B:2, C:4, D:3, E:5}
    PQ    = [(4, C), (5, E)]

    Settled: {A, B, D}

STEP 4: Process C (dist=4)
    Relax C→D: dist[D] = min(3, 4+3) = 3  ✗ no improvement (7 > 3)

    dist  = {A:0, B:2, C:4, D:3, E:5}
    PQ    = [(5, E)]

    Settled: {A, B, D, C}

STEP 5: Process E (dist=5)
    No outgoing edges.

    dist  = {A:0, B:2, C:4, D:3, E:5}
    PQ    = []  ← empty, DONE!

    Settled: {A, B, D, C, E}
```

**Final shortest distances from A:**

```
    A: 0   (source)
    B: 2   (A → B)
    C: 4   (A → C)
    D: 3   (A → B → D)
    E: 5   (A → B → D → E  or  A → B → E)
```

**Shortest path tree:**

```
    A ──2──→ B ──1──→ D ──2──→ E
    |
    4
    |
    v
    C

    Every node's shortest path from A follows these edges.
```

## 🔹 Complexity

| Metric | With Binary Heap (std) | Why |
|--------|----------------------|-----|
| **Time** | **O((V + E) log V)** | Each vertex extracted from PQ once: O(V log V). Each edge may trigger a PQ insertion: O(E log V). |
| **Space** | **O(V + E)** | Graph storage + priority queue + distance array. |

See [[Big O - Definition]].

```
Comparison:
    BFS (unweighted):     O(V + E)          ← faster but no weights
    Dijkstra (weighted):  O((V + E) log V)  ← handles weights
    Bellman-Ford:         O(V * E)          ← handles negative weights
```

## 🔹 Why It FAILS with Negative Weights

Dijkstra's assumes: once a node is "settled" (removed from PQ), its distance is final. Negative edges can violate this.

```
Counterexample:
        1
    A ────→ B
    |       |
   3|      -5    ← NEGATIVE EDGE!
    |       |
    v       v
    C       D

Dijkstra processes A(0), then B(1) since 1 < 3.
Settles B with dist=1, then D with dist = 1 + (-5) = -4.
Then settles C with dist=3.

BUT: the path A→C→...→D might be shorter!
Once B is settled, Dijkstra never reconsiders it.

With negative weights → use Bellman-Ford algorithm.
```

## 🔹 Template Code

```cpp
// Dijkstra's shortest path
// adj[u] = vector of {weight, v}  (adjacency list with weights)
vector<int> dijkstra(vector<vector<pair<int,int>>>& adj, int src) {
    int n = adj.size();
    vector<int> dist(n, INT_MAX);
    // min-heap: {distance, node}
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    
    dist[src] = 0;
    pq.push({0, src});
    
    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        
        if (d > dist[u]) continue;  // skip outdated entries
        
        for (auto [w, v] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

**Usage:**

```cpp
// Build weighted directed graph with 5 nodes (0-4)
int n = 5;
vector<vector<pair<int,int>>> adj(n);

// adj[u].push_back({weight, v})  means edge u→v with given weight
adj[0].push_back({2, 1});  // A→B, weight 2
adj[0].push_back({4, 2});  // A→C, weight 4
adj[1].push_back({1, 3});  // B→D, weight 1
adj[1].push_back({3, 4});  // B→E, weight 3
adj[2].push_back({3, 3});  // C→D, weight 3
adj[3].push_back({2, 4});  // D→E, weight 2

vector<int> dist = dijkstra(adj, 0);
// dist[i] = shortest distance from node 0 to node i
// dist = {0, 2, 4, 3, 5}
```

**To reconstruct the actual path** (not just distances):

```cpp
vector<int> dijkstraWithPath(vector<vector<pair<int,int>>>& adj, int src, int dest) {
    int n = adj.size();
    vector<int> dist(n, INT_MAX);
    vector<int> parent(n, -1);  // track path
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    
    dist[src] = 0;
    pq.push({0, src});
    
    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        if (d > dist[u]) continue;
        
        for (auto [w, v] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                parent[v] = u;          // remember where we came from
                pq.push({dist[v], v});
            }
        }
    }
    
    // Reconstruct path from dest to src
    vector<int> path;
    for (int v = dest; v != -1; v = parent[v])
        path.push_back(v);
    reverse(path.begin(), path.end());
    return path;  // e.g., {0, 1, 3, 4} for A→B→D→E
}
```

## 🔹 The `if (d > dist[u]) continue` Check

This line is critical. Without it, you process stale (outdated) priority queue entries and waste time.

```
Why stale entries exist:

    Node B starts with dist=10, pushed as (10, B)
    Later, B gets relaxed to dist=5, pushed as (5, B)
    
    PQ now has BOTH (5, B) and (10, B)
    
    When we pop (5, B): process it, update neighbors
    When we pop (10, B): d=10 > dist[B]=5 → SKIP! (stale)

Without this check: we'd re-process B and its neighbors
    → still correct, but much slower
    → could degrade to O(V * E) in worst case
```

## 🔹 Common Pitfalls

**1. Using with negative edge weights**

Dijkstra gives wrong answers with negative edges. Use Bellman-Ford instead.

**2. Forgetting the stale entry check**

```cpp
// ALWAYS include this:
if (d > dist[u]) continue;

// Without it: correct but slow (processes stale entries)
```

**3. Confusing with BFS**

```
BFS:      unweighted shortest path (counts edges)
Dijkstra: weighted shortest path   (sums weights)

Don't use BFS on weighted graphs.
Don't use Dijkstra on unweighted graphs (BFS is simpler + faster).
```

**4. Wrong pair ordering in adjacency list**

```cpp
// Be consistent! Pick one convention and stick with it:
// Convention A: adj[u] = {weight, neighbor}
// Convention B: adj[u] = {neighbor, weight}
// The template above uses Convention A: {weight, v}
```

## 🔹 When to Use What

| Graph Type | Algorithm | Time |
|------------|-----------|------|
| Unweighted | [[BFS - Breadth First Search]] | O(V + E) |
| Non-negative weights | **Dijkstra** | O((V + E) log V) |
| Negative weights (no neg. cycles) | Bellman-Ford | O(V * E) |
| All pairs shortest path | Floyd-Warshall | O(V^3) |

---

**See also:** [[Priority Queue]], [[BFS - Breadth First Search]], [[Graph Representations]]
