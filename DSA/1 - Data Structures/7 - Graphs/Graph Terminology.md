---
tags:
  - algorithms
  - data-structure
  - graph
---

## 🔹 Vertex (Node) and Edge

A **vertex** (or node) is a fundamental unit in a graph. An **edge** is a connection between two vertices. Together, they form a graph G = (V, E) where V is the set of vertices and E is the set of edges.

```
  Vertices: {A, B, C, D}

     (A)-----(B)
      |       |
      |       |
     (C)-----(D)

  Edges: {(A,B), (A,C), (B,D), (C,D)}
```

A graph is nothing more than dots (vertices) connected by lines (edges). Everything else in graph theory is built on this foundation.

---

## 🔹 Directed vs Undirected Graphs

### Undirected Graph

Edges have **no direction**. If A connects to B, then B also connects to A. Think of a two-way road or a friendship (if I'm your friend, you're mine).

```
  Undirected Graph:

     (A)-----(B)
      |       |
      |       |
     (C)-----(D)

  Edge (A,B) = Edge (B,A)
  You can traverse in either direction.
```

### Directed Graph (Digraph)

Edges have a **direction**. An edge from A to B does NOT imply an edge from B to A. Think of a one-way street or a Twitter "follow" (I can follow you without you following me).

```
  Directed Graph:

     (A)----->(B)
      |         |
      |         |
      v         v
     (C)<------(D)

  Edge (A,B) != Edge (B,A)
  Arrow indicates the allowed direction of traversal.
  A can reach B, but B cannot directly reach A.
```

In adjacency list representation:
- **Undirected**: add the edge in both directions (`adj[u].add(v)` AND `adj[v].add(u)`).
- **Directed**: add the edge in one direction only (`adj[u].add(v)`).

---

## 🔹 Weighted vs Unweighted Edges

### Unweighted

All edges are treated equally. The "cost" of traversing any edge is the same (typically 1). BFS finds shortest paths in unweighted graphs.

```
  Unweighted:

     (A)-----(B)
      |       |
     (C)-----(D)

  All edges have implicit weight of 1.
  Shortest path A->D: A->B->D (2 edges)
                  or: A->C->D (2 edges)
```

### Weighted

Each edge carries a numeric **weight** representing cost, distance, time, capacity, etc. Dijkstra or Bellman-Ford find shortest paths in weighted graphs.

```
  Weighted:

         3
  (A)---------(B)
   |            |
 1 |            | 8
   |            |
  (C)---------(D)
         2

  Shortest path A->D: A->C->D (cost 1+2 = 3)
  NOT A->B->D (cost 3+8 = 11)
```

---

## 🔹 Degree

The **degree** of a vertex is the number of edges connected to it.

### Undirected Graph Degree

```
     (A)-----(B)
      |     / |
      |   /   |
     (C)/    (D)

  degree(A) = 2  (edges to B, C)
  degree(B) = 3  (edges to A, C, D)
  degree(C) = 2  (edges to A, B)
  degree(D) = 1  (edge to B)
```

**Handshaking Lemma**: The sum of all vertex degrees = 2 * |E|.
In the graph above: 2 + 3 + 2 + 1 = 8 = 2 * 4 edges.

### Directed Graph: In-Degree and Out-Degree

For directed graphs, each vertex has two degree counts:
- **In-degree**: number of edges coming **into** the vertex.
- **Out-degree**: number of edges going **out of** the vertex.

```
     (A)----->(B)
      |         |
      |         |
      v         v
     (C)<------(D)----->(E)

  Vertex | In-degree | Out-degree
  -------+-----------+-----------
    A    |     0     |     2      (A->B, A->C)
    B    |     1     |     1      (A->B in, B->D out)
    C    |     2     |     0      (A->C, D->C in)
    D    |     1     |     2      (B->D in, D->C, D->E out)
    E    |     1     |     0      (D->E in)
```

Key facts:
- Sum of all in-degrees = Sum of all out-degrees = |E|.
- A vertex with in-degree 0 is a **source** (has no prerequisites).
- A vertex with out-degree 0 is a **sink** (nothing depends on it).
- Sources are starting points for topological sort.

---

## 🔹 Path

A **path** is a sequence of vertices where each consecutive pair is connected by an edge.

```
  Graph:
     (A)-----(B)-----(C)
      |               |
      |               |
     (D)-----(E)-----(F)

  Path from A to F:
    A -> B -> C -> F       (length 3)
    A -> D -> E -> F       (length 3)
    A -> B -> C -> F       (via top)
```

### Simple Path

A path where **no vertex is repeated**. Most algorithms assume simple paths.

```
  Simple path:     A -> B -> C -> F       (each vertex visited once)
  Non-simple path: A -> B -> C -> B -> ...  (B visited twice -- NOT simple)
```

### Shortest Path

The path with minimum total weight (weighted) or minimum number of edges (unweighted) between two vertices.

```
         1       10
  (A)-------(B)-------(D)
   |                    ^
   |  2        3        |
   +-------(C)---------+

  Shortest path A->D:
    A->B->D = 1 + 10 = 11
    A->C->D = 2 + 3  = 5   <-- shortest
```

Algorithms:
- **Unweighted**: BFS (O(V + E))
- **Non-negative weights**: Dijkstra (O((V + E) log V))
- **Negative weights (no neg cycles)**: Bellman-Ford (O(V * E))
- **All pairs**: Floyd-Warshall (O(V^3))

---

## 🔹 Cycle

A **cycle** is a path that starts and ends at the same vertex, with at least one edge. A graph with no cycles is called **acyclic**.

### Graph with a Cycle

```
     (A)-----(B)
      |       |
      |       |    <-- edges A-B, B-C, C-A form a cycle
      +-(C)---+
      
  Cycle: A -> B -> C -> A
```

### Acyclic Graph (No Cycles)

```
     (A)-----(B)
      |
      |            <-- no way to return to a visited vertex
     (C)-----(D)       without reusing an edge
```

### Directed Cycle vs Undirected Cycle

In directed graphs, all edges in the cycle must follow the same direction:

```
  Has a directed cycle:         No directed cycle (DAG):

     (A)----->(B)                  (A)----->(B)
      ^        |                    |         |
      |        |                    |         |
      +---(C)<-+                    v         v
                                   (C)       (D)
  Cycle: A->B->C->A
```

**Why cycles matter**:
- Cycles in a dependency graph mean **circular dependencies** (deadlock, infinite loop).
- Detecting cycles is needed for topological sort (only works on DAGs).
- Negative-weight cycles make shortest paths undefined (Bellman-Ford detects these).

---

## 🔹 Connected Components

A **connected component** is a maximal set of vertices such that there is a path between every pair of vertices in the set. A graph can have multiple disconnected components.

```
  This graph has 3 connected components:

  Component 1:         Component 2:     Component 3:
     (A)-----(B)         (E)-----(F)       (H)
      |       |                              |
     (C)-----(D)         (G)               (I)

  No edges connect the three components.
  A can reach B, C, D but cannot reach E, F, G, H, or I.
```

Finding connected components:
- **Undirected graph**: Run BFS/DFS from each unvisited vertex. Each run discovers one component.
- **Directed graph**: Use Tarjan's or Kosaraju's algorithm to find **Strongly Connected Components** (SCCs) where every vertex can reach every other vertex following edge directions.

```
  Strongly Connected Components (directed):

     (A)----->(B)<----+
      |        |      |
      v        v      |
     (C)----->(D)-----+

  SCC: {A, B, C, D} -- every vertex can reach every other
       via directed paths.

     (A)----->(B)         (E)----->(F)
      |        ^
      v        |
     (C)------+

  SCC 1: {A, B, C}  (A->C->B->... no, A->B, A->C->... 
                      check: B can reach A via B->?
                      Actually B has no outgoing to A)

  Let's fix:
     (A)----->(B)
      ^        |
      |        v
     (C)<-----(D)

  SCC: {A, B, D, C}
  Path: A->B->D->C->A (cycle through all)
```

---

## 🔹 DAG -- Directed Acyclic Graph

A **DAG** is a directed graph with **no cycles**. DAGs are extremely important because they model dependencies, scheduling, and hierarchies.

```
  DAG (valid topological ordering exists):

     (A)----->(B)----->(D)
      |                  ^
      |                  |
      +------>(C)-------+

  Topological order: A, B, C, D  (or A, C, B, D)
  Every edge goes "forward" in the ordering.
```

```
  NOT a DAG (has a cycle):

     (A)----->(B)
      ^        |
      |        v
     (D)<-----(C)

  Cycle: A->B->C->D->A
  No valid topological ordering exists.
```

### Why DAGs Matter

1. **Topological sort**: Order vertices so all edges point forward. Only possible on DAGs. Used for task scheduling, build systems (Makefile), course prerequisites.
2. **Dependency resolution**: Package managers (npm, pip) model dependencies as a DAG. A cycle means an impossible dependency.
3. **Dynamic programming on graphs**: Many DP problems can be modeled as shortest/longest path on a DAG (O(V + E) -- no need for Dijkstra).
4. **Critical path analysis**: Longest path in a DAG gives the minimum project completion time.

---

## 🔹 Bipartite Graph

A graph is **bipartite** if its vertices can be divided into two disjoint sets U and V such that every edge connects a vertex in U to a vertex in V. Equivalently, the graph is **2-colorable** -- you can color every vertex with one of two colors such that no two adjacent vertices share the same color.

```
  Bipartite Graph:

  Set U: {1, 3, 5}       Set V: {2, 4, 6}
  (color: RED)            (color: BLUE)

     [1]------[2]
      |        |
     [3]------[4]
      |        |
     [5]------[6]

  Every edge crosses from U to V.
  No edge within U or within V.
```

```
  NOT Bipartite (contains an odd-length cycle):

     [A]------[B]
      |        |
      +--[C]---+

  Cycle A-B-C-A has length 3 (odd).
  Try coloring: A=RED, B=BLUE, C=RED... but C is adjacent
  to A and both are RED. Contradiction.
```

### Testing for Bipartiteness

Run BFS/DFS and try to 2-color the graph. If you find an edge where both endpoints have the same color, it is not bipartite.

### When Bipartite Graphs Matter

- **Matching problems**: job assignments, marriage problem (Hungarian algorithm).
- **Graph coloring**: bipartite graphs need only 2 colors.
- **Many tree problems**: every tree is bipartite.

---

## 🔹 Dense vs Sparse Graphs

The density of a graph describes the ratio of actual edges to the maximum possible edges.

- **Maximum edges** (directed): V * (V - 1)
- **Maximum edges** (undirected): V * (V - 1) / 2

### Sparse Graph (E is close to V)

```
  Sparse: V=6, E=5  (max possible = 15)

     (0)-----(1)
              |
     (2)     (3)-----(4)
              |
             (5)

  Edge density: 5/15 = 33%
  Most pairs of vertices are NOT connected.
  Use: Adjacency List
```

### Dense Graph (E is close to V^2)

```
  Dense: V=4, E=5  (max possible = 6)

     (0)-----(1)
      | \   / |
      |  \ /  |
      |   X   |
      |  / \  |
     (2)-----(3)

  Edge density: 5/6 = 83%
  Most pairs of vertices ARE connected.
  Use: Adjacency Matrix may be appropriate
```

**Rules of thumb**:
- E < V * log(V) --> sparse
- E > V^2 / 2 --> dense
- Most real-world graphs (social networks, web graphs, road networks) are **sparse**.

---

## 🔹 Complete Graph (K_n)

A **complete graph** K_n is a graph where **every** vertex is connected to **every** other vertex. It is the densest possible simple graph.

```
  K_4 (Complete graph with 4 vertices):

     (A)-----(B)
      | \   / |
      |  \ /  |
      |   X   |
      |  / \  |
     (C)-----(D)

  Edges: AB, AC, AD, BC, BD, CD = 6 edges
  Formula: E = V * (V-1) / 2 = 4 * 3 / 2 = 6
```

```
  K_5 (Complete graph with 5 vertices):

        (A)
       / | \
      /  |  \
    (B)--+--(E)
     |\ /\ /|
     | X  X  |
     |/ \/ \|
    (C)----(D)

  Edges: 5 * 4 / 2 = 10 edges
```

Properties:
- Every vertex has degree V - 1.
- K_n has exactly V * (V - 1) / 2 edges.
- K_n is always (V-1)-colorable (chromatic number = V).
- Used as worst-case analysis baseline for graph algorithms.

---

## 🔹 Tree as a Special Case of Graph

A **tree** is a connected, acyclic undirected graph. It is the minimally connected graph -- removing any edge disconnects it.

```
  Tree (V=7, E=6):

          (A)
         / | \
       (B)(C)(D)
       /       \
     (E)       (F)
     /
   (G)

  Properties:
  - Connected: every vertex is reachable from every other
  - Acyclic: no cycles exist
  - E = V - 1 = 6 edges
  - Exactly one path between any two vertices
```

### Three Equivalent Definitions

A graph G with V vertices is a tree if and only if ANY TWO of these hold:
1. G is **connected**.
2. G is **acyclic**.
3. G has exactly **V - 1 edges**.

(Any two imply the third.)

```
  Connected + V-1 edges --> must be acyclic (tree)
  
  Connected, acyclic, V-1 edges:
          (1)                  NOT a tree (has a cycle):
         / \                        (1)
       (2) (3)                     / | \
       /     \                   (2)-(3)
     (4)     (5)                  \   /
                                   (4)
  E = 4 = V - 1 = 5 - 1          E = 5, V = 4 --> E > V-1
                                  Has cycle: 1-2-3-1
```

### Why Trees Matter

- Trees are the simplest connected structures -- they model hierarchies (file systems, org charts, XML/HTML DOM).
- Many graph problems have simpler solutions on trees (DP on trees, LCA, Euler tour).
- Spanning trees: a tree that includes all vertices of a graph. MST = Minimum Spanning Tree.
- Every connected graph has at least one spanning tree. If the graph has V vertices, the spanning tree has exactly V - 1 edges.

---

## 🔹 Summary Table

| Term | One-Line Definition | When It Matters |
|------|---------------------|-----------------|
| **Vertex / Edge** | A node and a connection between nodes | Foundation of every graph problem |
| **Directed** | Edges have a direction (one-way) | Modeling flows, dependencies, web links |
| **Undirected** | Edges go both ways | Friendships, roads, connections |
| **Weighted** | Edges carry numeric values | Shortest path, MST, network flow |
| **Unweighted** | All edges have equal cost | BFS-based shortest path |
| **Degree** | Number of edges at a vertex | Identifying hubs, Euler path conditions |
| **In-degree / Out-degree** | Edges in / edges out (directed) | Topological sort sources/sinks |
| **Path** | Sequence of connected vertices | Reachability, shortest path |
| **Cycle** | Path that returns to its start | Deadlock detection, DAG validation |
| **Connected Component** | Maximal reachable subgraph | Counting clusters, network analysis |
| **DAG** | Directed graph with no cycles | Topological sort, DP, scheduling |
| **Bipartite** | 2-colorable, no odd cycles | Matching, assignment problems |
| **Sparse** | E close to V | Use adjacency list, Dijkstra |
| **Dense** | E close to V^2 | Use adjacency matrix, Floyd-Warshall |
| **Complete (K_n)** | Every vertex connected to every other | Worst-case analysis, clique problems |
| **Tree** | Connected + acyclic + V-1 edges | Hierarchies, spanning trees, DP on trees |

---

## 🔹 Related Notes

- [[Graph Representations]] -- how to store graphs in code (adjacency matrix, adjacency list, edge list)
