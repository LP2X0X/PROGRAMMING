---
tags:
  - algorithms
  - data-structure
  - tree
---

## 🔹 Real-World Analogy

A tree is like a family tree or an organizational chart. There is one person at the top (the root), and each person can have children below them. You can trace a path from any person back up to the root, and no path ever loops back on itself. A binary tree is a family where every person has at most two children — a "left" child and a "right" child.

## 🔹 What Is a Tree?

A tree is a hierarchical, non-linear data structure made of **nodes** connected by **edges**. It has the following properties:

- There is exactly one **root** node (no parent).
- Every other node has exactly one parent.
- There are **no cycles** — you cannot follow edges from a node and arrive back at the same node.
- There is exactly one path between any two nodes.

A tree with `n` nodes always has exactly `n - 1` edges.

## 🔹 Terminology

| Term | Definition |
|---|---|
| **Root** | The topmost node with no parent |
| **Parent** | A node that has one or more children below it |
| **Child** | A node directly connected below another node |
| **Leaf** | A node with no children (also called an external node) |
| **Internal node** | A node that has at least one child |
| **Sibling** | Nodes that share the same parent |
| **Subtree** | A node and all of its descendants — itself a valid tree |
| **Edge** | The connection between a parent and a child |
| **Path** | A sequence of nodes connected by edges from one node to another |
| **Ancestor** | Any node on the path from a given node up to the root |
| **Descendant** | Any node in the subtree rooted at a given node |

## 🔹 Height vs Depth

These two terms are commonly confused. They measure in **opposite directions**.

```
Depth 0        -->        [A]           <-- Height 3
                         /   \
Depth 1        -->     [B]   [C]        <-- Height 2
                      /   \
Depth 2        -->  [D]   [E]           <-- Height 1
                   /
Depth 3        --> [F]                  <-- Height 0
```

| Concept | Measured From | Direction |
|---|---|---|
| **Depth of a node** | Root down to the node | Top-down |
| **Height of a node** | The node down to its deepest leaf | Bottom-up |
| **Height of the tree** | Root down to the deepest leaf | Top-down (equals root's height) |

- The **depth** of the root is always 0.
- The **height** of a leaf is always 0.
- The **height of the tree** = height of the root node = depth of the deepest leaf.

## 🔹 What Is a Binary Tree?

A **binary tree** is a tree where each node has **at most 2 children**, referred to as the **left child** and **right child**.

### Node Structure

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

## 🔹 Types of Binary Trees

### Full (Strict) Binary Tree

Every node has **0 or 2** children. No node has exactly one child.

```
        [1]
       /   \
     [2]   [3]
    /   \
  [4]   [5]
```

### Complete Binary Tree

All levels are fully filled **except possibly the last**, which is filled from **left to right** with no gaps.

```
        [1]
       /   \
     [2]   [3]
    /   \   /
  [4]  [5][6]
```

This is the structure used by **heaps** (binary heaps are always complete binary trees stored in an array).

### Perfect Binary Tree

All internal nodes have exactly 2 children, and **all leaves are at the same level**. Every level is completely filled.

```
        [1]
       /   \
     [2]   [3]
    / \    / \
  [4] [5][6] [7]
```

A perfect binary tree is both full and complete.

### Balanced Binary Tree

The height of the left and right subtrees of **every** node differ by at most 1. This guarantees O(log n) height.

```
        [1]
       /   \
     [2]   [3]        Balanced (height diff <= 1 at every node)
    /   \
  [4]   [5]
```

See [[Balanced BST]] for AVL and Red-Black tree details.

### Degenerate (Skewed) Binary Tree

Every internal node has exactly **one** child. The tree degenerates into a linked list.

```
  [1]                    [1]
    \                   /
    [2]              [2]
      \             /
      [3]        [3]
        \       /
        [4]  [4]

  Right-skewed    Left-skewed
```

This is the **worst case** for binary search trees — all operations become O(n).

## 🔹 Summary of Tree Types

| Type | Property | All leaves same level? |
|---|---|---|
| Full | Every node has 0 or 2 children | No |
| Complete | All levels filled except last (left-filled) | No |
| Perfect | All internal nodes have 2 children, all leaves same level | Yes |
| Balanced | Height difference at every node <= 1 | No |
| Degenerate | Every node has at most 1 child | No |

## 🔹 Mathematical Properties

These properties come up frequently in analysis and interviews.

| Property | Formula |
|---|---|
| Max nodes at level `k` | `2^k` |
| Max nodes in tree of height `h` | `2^(h+1) - 1` |
| Min height for `n` nodes | `floor(log2(n))` |
| Max height for `n` nodes | `n - 1` (degenerate) |
| Number of leaves in a perfect tree of height `h` | `2^h` |
| Number of internal nodes in a perfect tree | `2^h - 1` |
| In any binary tree: `L = I + 1` | Leaves = Internal nodes + 1 (for full binary trees) |

Where:
- `L` = number of leaf nodes
- `I` = number of internal nodes (nodes with at least one child)

The last property (`L = I + 1`) holds specifically for **full** binary trees.

### Derivation Example

For a **perfect binary tree** of height `h = 3`:

```
Level 0:  1 node    (2^0)          [1]
Level 1:  2 nodes   (2^1)        /     \
Level 2:  4 nodes   (2^2)      [2]     [3]
Level 3:  8 nodes   (2^3)     / \      / \
                             [4] [5] [6] [7]
Total = 1+2+4+8 = 15        / \  / \ / \ / \
     = 2^(3+1) - 1          8  9 ...      15
     = 2^4 - 1 = 15
```

## 🔹 Representing Binary Trees

### Pointer-Based (Linked)

Each node stores pointers to its left and right children. This is the standard representation used in most interview problems.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
};
```

### Array-Based (Implicit)

For a **complete** binary tree, you can store nodes in an array level by level. Given a node at index `i` (0-indexed):

| Relationship | Index |
|---|---|
| Left child | `2*i + 1` |
| Right child | `2*i + 2` |
| Parent | `(i - 1) / 2` |

```
Array:  [1, 2, 3, 4, 5, 6, 7]
Index:   0  1  2  3  4  5  6

Tree:
          [1]        index 0
         /   \
       [2]   [3]     index 1, 2
      / \    / \
    [4] [5][6] [7]   index 3, 4, 5, 6
```

This is exactly how a **binary heap** works. See [[Heap]] for more details.

## 🔹 Common Pitfalls

1. **Confusing height and depth.** Height is bottom-up (leaf = 0), depth is top-down (root = 0). The height of the tree equals the depth of its deepest node.

2. **Confusing full vs complete vs perfect.** A perfect tree is both full and complete, but full does not imply complete and complete does not imply full.

3. **Forgetting the degenerate case.** When analyzing BST operations, always consider the worst case: a skewed tree that behaves like a linked list with O(n) operations.

4. **Off-by-one in height calculations.** Some textbooks define height as number of edges (root-only tree has height 0), others as number of nodes on the longest path (root-only tree has height 1). LeetCode and most interview contexts use the **edge-based** definition.

5. **Assuming trees are always balanced.** Unless stated otherwise (AVL, Red-Black), a binary tree or BST can be unbalanced.

## 🔹 Key Interview Insight

Most tree problems are solved with **recursion**. The pattern is almost always:

1. Handle the base case (`node == nullptr`).
2. Do something with the current node.
3. Recurse on `left` and/or `right` subtrees.
4. Combine results.

```cpp
int solve(TreeNode* node) {
    if (!node) return base_case;          // 1. base case

    int leftResult  = solve(node->left);  // 3. recurse left
    int rightResult = solve(node->right); // 3. recurse right

    return combine(node->val, leftResult, rightResult);  // 4. combine
}
```

This structure applies to finding height, counting nodes, checking balance, mirroring, path sums, and dozens of other classic problems.

See [[Tree Traversals]] for the specific patterns (in-order, pre-order, post-order, level-order).
See [[Binary Search Tree]] for the ordered variant with search/insert/delete.
