---
tags:
  - algorithms
  - data-structure
  - tree
  - balanced
---

## 🔹 Real-World Analogy

Think of a balanced BST like a well-organized filing cabinet. If you throw files in randomly, some drawers get overstuffed (skewed tree) and finding anything takes forever. A balanced BST is like having a system that automatically redistributes files across drawers whenever things get lopsided, so every lookup stays fast.

## 🔹 Why Balance Matters

A [[Binary Search Tree]] promises O(log n) operations on average, but the worst case is O(n) when the tree is skewed (degenerate). Inserting sorted data into a plain BST produces exactly this worst case:

```
Insert 1, 2, 3, 4, 5 into a plain BST:

    [1]                    Balanced BST with same data:
      \
      [2]                        [3]
        \                       /   \
        [3]                   [2]   [4]
          \                  /         \
          [4]              [1]         [5]
            \
            [5]

  Height = 4 (O(n))        Height = 2 (O(log n))
  Search for 5: 5 steps    Search for 5: 2 steps
```

A **balanced BST** restructures itself after insertions and deletions to guarantee that the height stays O(log n), preserving efficient operations.

## 🔹 What "Balanced" Means

A binary tree is **height-balanced** if, for every node, the heights of its left and right subtrees differ by at most 1.

```
        [10]
       /    \
     [5]    [15]         Left height = 2, Right height = 1
    / \        \         Difference = 1 --> balanced at root
  [3] [7]     [20]
  /
[1]                      But check node [5]: left height=1, right height=0
                         Difference = 1 --> balanced

                         Every node satisfies |leftH - rightH| <= 1
                         --> This tree IS balanced.
```

```
        [10]
       /
     [5]                 Left height = 2, Right height = 0
    /                    Difference = 2 --> NOT balanced!
  [3]
```

## 🔹 Self-Balancing BST Variants

| Tree Type | Balance Guarantee | Strictness | Use Case |
|---|---|---|---|
| **AVL Tree** | Height diff <= 1 at every node | Strictly balanced | Lookup-heavy workloads |
| **Red-Black Tree** | No path is more than 2x longest | Loosely balanced | General-purpose (std::map) |
| **B-Tree** | All leaves at same depth | Perfectly balanced | Databases, file systems |
| **Splay Tree** | Amortized O(log n) | No strict balance | Caching, recently accessed data |

For interviews, **AVL trees** and **Red-Black trees** are the two you need to understand. You rarely need to implement them from scratch, but you must understand their guarantees and when to use them.

---

## 🔹 AVL Trees

Named after Adelson-Velsky and Landis (1962), the AVL tree is the first self-balancing BST ever invented.

### Balance Factor

Every node stores a **balance factor**:

```
balance_factor(node) = height(left subtree) - height(right subtree)
```

A valid AVL tree has `balance_factor` in `{-1, 0, 1}` for **every** node.

```
        [30]  bf=1
       /    \
     [20]   [40]  bf=0
    /    \
  [10]  [25]
  bf=0  bf=0

  bf(30) = height(left=2) - height(right=1) = 1   OK
  bf(20) = height(left=1) - height(right=1) = 0   OK
  bf(40) = 0 - 0 = 0                               OK
```

### When Imbalance Occurs

After an insertion or deletion, if any node's balance factor becomes `+2` or `-2`, the tree is imbalanced and needs **rotations** to fix it.

There are exactly **four cases**, fixed by **two types of rotations**:

| Imbalance Case | Balance Factor Pattern | Fix |
|---|---|---|
| Left-Left (LL) | Node = +2, Left child = +1 or 0 | Single Right Rotation |
| Right-Right (RR) | Node = -2, Right child = -1 or 0 | Single Left Rotation |
| Left-Right (LR) | Node = +2, Left child = -1 | Left Rotate child, then Right Rotate node |
| Right-Left (RL) | Node = -2, Right child = +1 | Right Rotate child, then Left Rotate node |

### Single Right Rotation (LL Case)

When a node is left-heavy (bf = +2) and its left child is left-heavy or balanced (bf = +1 or 0).

```
Before (LL imbalance at [30]):

        [30]  bf=+2
       /
     [20]  bf=+1
    /
  [10]  bf=0

Right Rotate around [30]:

Step 1: [20] becomes the new root of this subtree
Step 2: [30] becomes [20]'s right child
Step 3: [20]'s old right child (if any) becomes [30]'s left child

        [20]  bf=0
       /    \
     [10]  [30]
     bf=0   bf=0
```

With a more complex subtree:

```
Before:
          [30]  bf=+2
         /    \
       [20]   [T4]
      /    \
    [10]   [T3]
   /    \
 [T1]  [T2]

After Right Rotate around [30]:

          [20]
         /    \
       [10]   [30]
      /    \  /    \
    [T1] [T2][T3] [T4]
```

### Single Left Rotation (RR Case)

Mirror image of the right rotation. When a node is right-heavy (bf = -2) and its right child is right-heavy or balanced.

```
Before (RR imbalance at [10]):

  [10]  bf=-2
     \
     [20]  bf=-1
        \
        [30]  bf=0

Left Rotate around [10]:

        [20]  bf=0
       /    \
     [10]  [30]
     bf=0   bf=0
```

With a more complex subtree:

```
Before:
    [10]  bf=-2
   /    \
 [T1]  [20]
       /    \
     [T2]  [30]
           /    \
         [T3]  [T4]

After Left Rotate around [10]:

          [20]
         /    \
       [10]   [30]
      /    \  /    \
    [T1] [T2][T3] [T4]
```

### Rotation Code

```cpp
TreeNode* rightRotate(TreeNode* y) {
    TreeNode* x = y->left;
    TreeNode* T2 = x->right;

    // Perform rotation
    x->right = y;
    y->left = T2;

    // Update heights (assuming height field exists)
    y->height = 1 + max(height(y->left), height(y->right));
    x->height = 1 + max(height(x->left), height(x->right));

    return x;  // new root of this subtree
}

TreeNode* leftRotate(TreeNode* x) {
    TreeNode* y = x->right;
    TreeNode* T2 = y->left;

    // Perform rotation
    y->left = x;
    x->right = T2;

    // Update heights
    x->height = 1 + max(height(x->left), height(x->right));
    y->height = 1 + max(height(y->left), height(y->right));

    return y;  // new root of this subtree
}
```

### Double Rotation: Left-Right (LR Case)

When a node is left-heavy (bf = +2) but its left child is right-heavy (bf = -1). A single right rotation would not fix this — the subtree would just flip to a right-heavy imbalance.

Solution: first left-rotate the left child, then right-rotate the node.

```
Before (LR imbalance at [30]):

        [30]  bf=+2
       /
     [10]  bf=-1
        \
        [20]  bf=0

Step 1: Left Rotate around [10]:

        [30]
       /
     [20]
    /
  [10]

Step 2: Right Rotate around [30]:

        [20]
       /    \
     [10]  [30]
```

### Double Rotation: Right-Left (RL Case)

Mirror of LR. Node is right-heavy, its right child is left-heavy.

```
Before (RL imbalance at [10]):

  [10]  bf=-2
     \
     [30]  bf=+1
    /
  [20]  bf=0

Step 1: Right Rotate around [30]:

  [10]
     \
     [20]
        \
        [30]

Step 2: Left Rotate around [10]:

        [20]
       /    \
     [10]  [30]
```

### AVL Insert (Pseudocode)

```cpp
TreeNode* insert(TreeNode* node, int key) {
    // 1. Standard BST insert
    if (!node) return new TreeNode(key);

    if (key < node->val)
        node->left = insert(node->left, key);
    else if (key > node->val)
        node->right = insert(node->right, key);
    else
        return node;  // no duplicates

    // 2. Update height
    node->height = 1 + max(height(node->left), height(node->right));

    // 3. Get balance factor
    int bf = getBalance(node);  // height(left) - height(right)

    // 4. Fix imbalance (4 cases)

    // Left-Left
    if (bf > 1 && key < node->left->val)
        return rightRotate(node);

    // Right-Right
    if (bf < -1 && key > node->right->val)
        return leftRotate(node);

    // Left-Right
    if (bf > 1 && key > node->left->val) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
    }

    // Right-Left
    if (bf < -1 && key < node->right->val) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
    }

    return node;  // balanced, no rotation needed
}
```

### AVL Tree Complexity

| Operation | Time | Space |
|---|---|---|
| Search | O(log n) guaranteed | O(1) iterative |
| Insert | O(log n) guaranteed | O(log n) recursive |
| Delete | O(log n) guaranteed | O(log n) recursive |
| Rotation | O(1) | O(1) |

AVL trees guarantee O(log n) because the height is always bounded:

```
h <= 1.44 * log2(n + 2) - 0.328
```

In practice, AVL trees are slightly shorter than Red-Black trees, making lookups slightly faster, but insertions/deletions may require more rotations.

---

## 🔹 Red-Black Trees

Red-Black trees are a more relaxed form of balanced BST. They are used in most standard library implementations (`std::map`, `std::set` in C++, `TreeMap` in Java).

### Properties (The Five Rules)

Every node is colored either **red** or **black**, and the tree satisfies:

1. Every node is either red or black.
2. The **root** is always **black**.
3. Every **leaf** (null/NIL node) is **black**.
4. If a node is **red**, both its children must be **black** (no two consecutive red nodes).
5. Every path from a node to any of its descendant NIL leaves has the **same number of black nodes** (called the **black-height**).

```
            [13 B]
           /      \
       [8 R]      [17 R]
      /    \      /     \
   [1 B] [11 B] [15 B] [25 B]
      \                 /
     [6 R]           [22 R]

B = black, R = red

Check property 5 from root:
  Path to [1]'s left NIL:  13(B) -> 8(R) -> 1(B) -> NIL  = 2 black nodes (not counting NIL)
  Path to [6]'s left NIL:  13(B) -> 8(R) -> 1(B) -> 6(R) -> NIL  = 2 black nodes
  Path to [22]'s right NIL: 13(B) -> 17(R) -> 25(B) -> 22(R) -> NIL  = 2 black nodes
  All paths have black-height = 2.  Valid!
```

### Why These Rules Guarantee Balance

Rule 5 ensures that no path is more than twice as long as any other (because the longest path can have alternating red-black nodes, while the shortest is all-black). This gives a height bound of:

```
h <= 2 * log2(n + 1)
```

Not as tight as AVL, but still O(log n).

### Red-Black vs AVL

| Aspect | AVL | Red-Black |
|---|---|---|
| Balance strictness | Strict (bf <= 1) | Relaxed (no path > 2x another) |
| Max height (for n nodes) | ~1.44 log n | ~2 log n |
| Lookup speed | Slightly faster (shorter) | Slightly slower |
| Insert/Delete | More rotations | Fewer rotations |
| Best for | Read-heavy workloads | Write-heavy or mixed workloads |
| Standard library use | Rarely | C++ std::map, Java TreeMap |

### Key Takeaway for Interviews

You almost never need to implement Red-Black tree rotations and recoloring in an interview. What you need to know:

1. Red-Black trees are self-balancing BSTs that guarantee O(log n) for all operations.
2. They are the backing data structure for `std::map`, `std::set` (C++), `TreeMap`, `TreeSet` (Java).
3. They are "good enough" balanced — slightly worse than AVL for lookups, but better for mixed insert/delete/lookup workloads.
4. When an interviewer says "assume you have a balanced BST," they typically mean something like a Red-Black tree.

---

## 🔹 When to Use Self-Balancing BSTs

Use a self-balancing BST (or its standard library wrapper) when you need:

- **Dynamic sorted data** with guaranteed O(log n) insert, delete, and search.
- **Ordered operations** like finding the kth smallest element, finding the closest value, range queries.
- **Sorted iteration** in O(n).

In practice, you use the standard library:

| Language | Sorted Set | Sorted Map | Underlying Structure |
|---|---|---|---|
| C++ | `std::set` | `std::map` | Red-Black Tree |
| Java | `TreeSet` | `TreeMap` | Red-Black Tree |
| C# | `SortedSet<T>` | `SortedDictionary<K,V>` | Red-Black Tree |
| Python | -- | -- | Use `sortedcontainers` (B-tree based) |

### When NOT to Use

- If you only need lookup/insert/delete without ordering, use a **hash table** (O(1) average vs O(log n)).
- If data is static (no insertions/deletions), use a **sorted array** with binary search (better cache performance, less memory).

## 🔹 Common Pitfalls

1. **Thinking you need to implement AVL/Red-Black from scratch in interviews.** You almost never do. Know the concept and guarantees. In coding interviews, `std::set`/`std::map` are your balanced BST.

2. **Forgetting to update heights after rotations.** In AVL tree implementations, the height of rotated nodes must be recalculated. Updating in the wrong order causes bugs.

3. **Confusing rotation direction.** A "right rotation" means the left child moves **up** and the node moves **down-right**. The name refers to the direction the parent node moves, not the child.

4. **Assuming all BSTs are balanced.** Unless explicitly stated (AVL, Red-Black, or "balanced BST"), a BST can be degenerate. Always clarify in interviews.

5. **Not recognizing when a problem needs a balanced BST.** If a problem requires maintaining sorted order with dynamic insertions and you need better than O(n) per operation, think `std::set`/`std::map`.

## 🔹 Complexity Summary

| Operation | AVL | Red-Black | Plain BST (worst) |
|---|---|---|---|
| Search | O(log n) | O(log n) | O(n) |
| Insert | O(log n) | O(log n) | O(n) |
| Delete | O(log n) | O(log n) | O(n) |
| Space | O(n) | O(n) | O(n) |
| Max Rotations (insert) | O(1) (at most 2) | O(1) (at most 2) | N/A |
| Max Rotations (delete) | O(log n) | O(1) (at most 3) | N/A |

Both AVL and Red-Black trees guarantee O(log n) for all primary operations, eliminating the O(n) worst case of plain BSTs. The trade-off is implementation complexity and a small constant-factor overhead for maintaining balance.
