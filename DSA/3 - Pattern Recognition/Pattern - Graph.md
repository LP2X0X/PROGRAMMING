---
tags:
  - algorithms
  - pattern-recognition
  - graph
---

## 🔹 When to Suspect This Pattern

- Problem involves **relationships or connections** between entities
- Keywords: "connected", "reachable", "shortest path", "cycle", "islands", "network"
- Input is a **grid** (grids are implicit graphs — each cell connects to neighbors)
- Problem involves **dependencies or prerequisites** (course schedule, build order)
- Need to explore **all possibilities in a connected structure**
- "Number of connected components" or "is there a path from A to B"

---

## 🔹 Confirming It's the Right Pattern

- [ ] Can the problem be modeled as **nodes and edges**?
- [ ] Are you exploring **connected** regions or finding **paths**?
- [ ] Is the input a grid, adjacency list, adjacency matrix, or edge list?
- [ ] Does the problem ask about **reachability**, **shortest distance**, or **ordering**?

---

## 🔹 BFS vs DFS Decision

| Question | BFS | DFS |
|---|---|---|
| **Shortest path** (unweighted)? | **Yes** — BFS guarantees it | No — DFS may find longer path first |
| **Level-by-level** processing? | **Yes** — natural level order | No |
| **Explore all / connectivity**? | Either | **Yes** — simpler with recursion |
| **Cycle detection** (directed)? | Topological sort (Kahn's) | **Yes** — with 3-color marking |
| **Topological sort**? | **Kahn's** (BFS-based) | **Yes** — reverse post-order |
| **Memory concern**? | O(width) queue | **O(depth) stack** — usually less |

---

## 🔹 Templates

### BFS (Shortest Path / Level Order)

```cpp
int bfs(vector<vector<int>>& graph, int start, int target) {
    queue<int> q;
    unordered_set<int> visited;
    q.push(start);
    visited.insert(start);
    int distance = 0;

    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            int node = q.front(); q.pop();
            if (node == target) return distance;
            for (int neighbor : graph[node]) {
                if (!visited.count(neighbor)) {
                    visited.insert(neighbor);
                    q.push(neighbor);
                }
            }
        }
        distance++;
    }
    return -1;  // not reachable
}
```

### DFS (Explore All / Connected Components)

```cpp
void dfs(vector<vector<int>>& graph, int node, vector<bool>& visited) {
    visited[node] = true;
    for (int neighbor : graph[node]) {
        if (!visited[neighbor])
            dfs(graph, neighbor, visited);
    }
}

// Count connected components
int count = 0;
vector<bool> visited(n, false);
for (int i = 0; i < n; i++) {
    if (!visited[i]) {
        dfs(graph, i, visited);
        count++;
    }
}
```

### Grid BFS/DFS (Number of Islands)

```cpp
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

void dfs(vector<vector<char>>& grid, int r, int c) {
    if (r < 0 || r >= grid.size() || c < 0 || c >= grid[0].size()) return;
    if (grid[r][c] != '1') return;
    grid[r][c] = '0';  // mark visited
    for (int d = 0; d < 4; d++)
        dfs(grid, r + dx[d], c + dy[d]);
}

int numIslands(vector<vector<char>>& grid) {
    int count = 0;
    for (int r = 0; r < grid.size(); r++)
        for (int c = 0; c < grid[0].size(); c++)
            if (grid[r][c] == '1') {
                dfs(grid, r, c);
                count++;
            }
    return count;
}
```

### Topological Sort (Kahn's BFS)

```cpp
vector<int> topoSort(int n, vector<vector<int>>& graph) {
    vector<int> inDegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : graph[u])
            inDegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0) q.push(i);

    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int neighbor : graph[node])
            if (--inDegree[neighbor] == 0)
                q.push(neighbor);
    }

    // If order.size() != n, there's a cycle (not a DAG)
    return order;
}
```

### Dijkstra (Weighted Shortest Path)

```cpp
vector<int> dijkstra(vector<vector<pair<int,int>>>& graph, int start) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;

    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;  // stale entry
        for (auto [v, w] : graph[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

---

## 🔹 Classic Problems

| Problem | Technique | Key Insight |
|---|---|---|
| **Number of Islands** | DFS/BFS on grid | Flood fill, count components |
| **Clone Graph** | BFS/DFS + Hash Map | Map old node → cloned node |
| **Course Schedule** | Topological Sort | Detect cycle in directed graph |
| **Word Ladder** | BFS | Each word is a node, one-letter change = edge |
| **Shortest Path in Grid** | BFS | Unweighted → BFS gives shortest |
| **Network Delay Time** | Dijkstra | Weighted shortest path from source |
| **Accounts Merge** | Union-Find or DFS | Group by connected emails |
| **Bipartite Check** | BFS/DFS coloring | Alternate colors; conflict = not bipartite |
| **Surrounded Regions** | DFS from border | Mark border-connected O's, flip the rest |

---

## 🔹 Graph Representation Cheat Sheet

| Format | When to Use | Space |
|---|---|---|
| **Adjacency List** | Sparse graphs (E << V^2) — most problems | O(V + E) |
| **Adjacency Matrix** | Dense graphs, Floyd-Warshall, quick edge check | O(V^2) |
| **Edge List** | Kruskal's, Bellman-Ford, union-find problems | O(E) |
| **Implicit Graph (Grid)** | Grid problems — no explicit graph needed | O(rows * cols) |

---

## 🔹 Common Mistakes

- **Forgetting visited check**: leads to infinite loops in cyclic graphs
- **Grid boundaries**: always check `0 <= r < rows && 0 <= c < cols` before accessing
- **Directed vs undirected**: undirected edges = add both directions. Topological sort only works on **directed** acyclic graphs
- **Dijkstra with negative weights**: doesn't work. Use Bellman-Ford instead
- **Topological sort cycle detection**: if result size < n, there's a cycle
- **BFS shortest path assumes uniform weight**: if weights vary, use Dijkstra, not BFS

---

## 🔹 Related Patterns

- [[Pattern - Tree]] — trees are acyclic connected graphs
- [[Pattern - Dynamic Programming]] — shortest path problems, graph DP
- [[Pattern - Backtracking]] — DFS-style exploration with backtracking
- [[How to Pick the Right Algorithm]] — choosing BFS vs DFS vs Dijkstra
