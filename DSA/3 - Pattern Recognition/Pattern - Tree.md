---
tags:
  - algorithms
  - pattern-recognition
  - tree
---

## 🔹 When to Suspect This Pattern

- Input is explicitly a **tree** (binary tree, BST, n-ary tree)
- **Hierarchical structure** — parent-child relationships
- Keywords: "root to leaf", "depth", "height", "level order", "ancestor", "subtree", "serialize"
- The problem has a **recursive structure** — solve for children, combine results
- Grid/string problem with a tree-shaped recursion (decision tree → often [[Pattern - Backtracking]])

---

## 🔹 Confirming It's the Right Pattern

- [ ] Is the input a tree (or can it be modeled as one)?
- [ ] Can the problem be solved by combining solutions from **left and right subtrees**?
- [ ] Is there a natural **base case** (null node, leaf node)?
- [ ] Does the answer depend on **depth, height, path, or level**?

---

## 🔹 Core Traversals

| Traversal | Order | Use Case |
|---|---|---|
| **Pre-order** | Root → Left → Right | Serialize tree, copy tree |
| **In-order** | Left → Root → Right | BST sorted order |
| **Post-order** | Left → Right → Root | Delete tree, compute heights, subtree aggregation |
| **Level-order** (BFS) | Level by level | Level averages, right side view, zigzag |

```cpp
// DFS traversal template (recursive)
void dfs(TreeNode* node) {
    if (!node) return;
    // PRE-ORDER: process node here
    dfs(node->left);
    // IN-ORDER: process node here
    dfs(node->right);
    // POST-ORDER: process node here
}

// BFS level-order template
void bfs(TreeNode* root) {
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int levelSize = q.size();
        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front(); q.pop();
            // process node
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
}
```

---

## 🔹 The Three DFS Return Patterns

### Pattern 1: Return a value up (post-order style)

Compute something from children, return to parent.

```cpp
// Max depth of tree
int maxDepth(TreeNode* node) {
    if (!node) return 0;
    return 1 + max(maxDepth(node->left), maxDepth(node->right));
}
```

**Use for**: height, diameter, balanced check, subtree sum.

### Pattern 2: Pass information down (pre-order style)

Carry state from parent to children via parameters.

```cpp
// Check if path sum equals target
bool hasPathSum(TreeNode* node, int remaining) {
    if (!node) return false;
    remaining -= node->val;
    if (!node->left && !node->right) return remaining == 0;
    return hasPathSum(node->left, remaining) ||
           hasPathSum(node->right, remaining);
}
```

**Use for**: root-to-leaf paths, prefix tracking, depth passing.

### Pattern 3: Global variable / reference update

Update an external answer during traversal.

```cpp
int diameter = 0;
int height(TreeNode* node) {
    if (!node) return 0;
    int L = height(node->left);
    int R = height(node->right);
    diameter = max(diameter, L + R);  // update global
    return 1 + max(L, R);
}
```

**Use for**: diameter, longest path, max path sum (where the "best" path may not go through root).

---

## 🔹 Classic Problems

| Problem | Traversal | Key Insight |
|---|---|---|
| **Max Depth** | Post-order | 1 + max(left, right) |
| **Validate BST** | In-order | In-order must be strictly increasing; or pass min/max bounds down |
| **Level Order Traversal** | BFS | Process `q.size()` nodes per level |
| **Lowest Common Ancestor** | Post-order | If both sides return non-null, current node is LCA |
| **Diameter of Binary Tree** | Post-order + global | Diameter at each node = left height + right height |
| **Serialize / Deserialize** | Pre-order | Record nulls as markers; rebuild with queue/index |
| **Symmetric Tree** | Recursive compare | Compare left.left with right.right AND left.right with right.left |
| **Path Sum** | Pre-order (passing remaining) | Subtract as you go, check at leaf |
| **Binary Tree Right Side View** | BFS | Last node of each level |

---

## 🔹 BST-Specific Patterns

| Property | How to Use |
|---|---|
| In-order = sorted | Validate BST, find Kth smallest, convert to sorted list |
| Left < Root < Right | Search in O(h), insert in O(h) |
| Floor/Ceiling | Modified search: track closest smaller/larger |

> [!tip] BST Validation
> Don't just check `node->left->val < node->val`. That only checks immediate children. You need to pass **min/max bounds** down the tree.
> ```cpp
> bool isValidBST(TreeNode* node, long minVal, long maxVal) {
>     if (!node) return true;
>     if (node->val <= minVal || node->val >= maxVal) return false;
>     return isValidBST(node->left, minVal, node->val) &&
>            isValidBST(node->right, node->val, maxVal);
> }
> ```

---

## 🔹 Common Mistakes

- **Forgetting null checks**: always handle `if (!node) return ...` as the base case
- **Confusing height and depth**: height = bottom-up (leaf = 0); depth = top-down (root = 0)
- **BST validation**: checking only parent-child, not entire subtree bounds
- **Not handling single-child nodes**: some problems behave differently at nodes with one vs two children
- **Assuming balanced**: unless stated, trees can be skewed (height = n, not log n)
- **Path sum confusion**: "path" can mean root-to-leaf, root-to-any, or any-to-any depending on the problem

---

## 🔹 Related Patterns

- [[Pattern - Graph]] — trees are special graphs (connected, acyclic)
- [[Pattern - Dynamic Programming]] — tree DP (optimize over subtrees)
- [[Pattern - Backtracking]] — tree-shaped recursion for decision trees
- [[Pattern - Binary Search]] — BST search is a form of binary search
