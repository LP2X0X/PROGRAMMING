---
tags:
  - algorithms
  - data-structure
  - bst
  - tree
---

## 🔹 Real-World Analogy

Imagine a dictionary. You do not start at page 1 and read every entry to find a word. You open roughly to the middle, check if your word comes before or after, then repeat in the correct half. A BST works the same way — at every node you make a binary decision (go left or go right), cutting the search space in half each time.

## 🔹 Definition

A **Binary Search Tree (BST)** is a [[Binary Tree]] where every node satisfies the **BST property**:

> For every node `N`:
> - All values in the **left** subtree are **less than** `N.val`
> - All values in the **right** subtree are **greater than** `N.val`

```
          [8]
         /   \
       [3]   [10]
      / \       \
    [1] [6]    [14]
       / \     /
     [4] [7] [13]
```

Verify: every node's left subtree has only smaller values, every node's right subtree has only larger values.

Important: the BST property applies to the **entire** subtree, not just the immediate children. A node's left child being smaller is not sufficient — every node in the left subtree must be smaller.

## 🔹 Why BSTs Matter

BSTs provide **ordered** storage with efficient search, insertion, and deletion — like having a sorted array that is also efficient to insert into.

| Operation | Sorted Array | Linked List | BST (balanced) | BST (worst) |
|---|---|---|---|---|
| Search | O(log n) | O(n) | O(log n) | O(n) |
| Insert | O(n) | O(1) at head | O(log n) | O(n) |
| Delete | O(n) | O(1) if at node | O(log n) | O(n) |
| Sorted traversal | O(n) | O(n log n) sort | O(n) | O(n) |

BSTs combine the search speed of sorted arrays with the insertion speed of linked lists — as long as they stay balanced.

## 🔹 Node Structure

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

## 🔹 Search

To find a value in a BST, start at the root and follow the BST property:

1. If the current node is null, the value is not in the tree.
2. If the value equals the current node, found it.
3. If the value is less, go left.
4. If the value is greater, go right.

### Step-by-Step: Search for 6

```
Step 1: Start at root [8]
        6 < 8 --> go LEFT

          [8]
         /
       [3]

Step 2: At node [3]
        6 > 3 --> go RIGHT

       [3]
          \
          [6]

Step 3: At node [6]
        6 == 6 --> FOUND!
```

### Code

```cpp
// Recursive
TreeNode* search(TreeNode* root, int target) {
    if (!root || root->val == target)
        return root;
    if (target < root->val)
        return search(root->left, target);
    else
        return search(root->right, target);
}

// Iterative (preferred — no stack overflow risk)
TreeNode* search(TreeNode* root, int target) {
    while (root && root->val != target) {
        if (target < root->val)
            root = root->left;
        else
            root = root->right;
    }
    return root;  // nullptr if not found
}
```

## 🔹 Insert

Insertion always adds the new node as a **leaf**. Walk down the tree using the BST property until you find a null spot.

### Step-by-Step: Insert 5

```
Starting tree:
          [8]
         /   \
       [3]   [10]
      / \
    [1] [6]

Step 1: 5 < 8 --> go LEFT to [3]
Step 2: 5 > 3 --> go RIGHT to [6]
Step 3: 5 < 6 --> go LEFT... it's NULL!
Step 4: Insert [5] as left child of [6]

Result:
          [8]
         /   \
       [3]   [10]
      / \
    [1] [6]
       /
     [5]    <-- new node
```

### Code

```cpp
// Recursive
TreeNode* insert(TreeNode* root, int val) {
    if (!root)
        return new TreeNode(val);

    if (val < root->val)
        root->left = insert(root->left, val);
    else if (val > root->val)
        root->right = insert(root->right, val);
    // val == root->val: duplicate, do nothing (or handle per requirements)

    return root;
}

// Iterative
TreeNode* insert(TreeNode* root, int val) {
    TreeNode* newNode = new TreeNode(val);
    if (!root) return newNode;

    TreeNode* curr = root;
    TreeNode* parent = nullptr;

    while (curr) {
        parent = curr;
        if (val < curr->val)
            curr = curr->left;
        else if (val > curr->val)
            curr = curr->right;
        else
            return root;  // duplicate
    }

    if (val < parent->val)
        parent->left = newNode;
    else
        parent->right = newNode;

    return root;
}
```

## 🔹 Finding Min and Max

Because of the BST property:
- The **minimum** value is the **leftmost** node (keep going left).
- The **maximum** value is the **rightmost** node (keep going right).

```
          [8]
         /   \
       [3]   [10]
      / \       \
    [1] [6]    [14]
    ^             ^
   MIN           MAX
```

```cpp
TreeNode* findMin(TreeNode* root) {
    while (root && root->left)
        root = root->left;
    return root;
}

TreeNode* findMax(TreeNode* root) {
    while (root && root->right)
        root = root->right;
    return root;
}
```

## 🔹 In-Order Successor and Predecessor

The **in-order successor** of a node is the node with the **smallest value greater than** the current node. The **in-order predecessor** is the node with the **largest value smaller than** the current node.

These follow directly from the BST property and the in-order traversal sequence. See [[Tree Traversals]] for more on in-order traversal.

### Finding In-Order Successor

Two cases:

**Case 1: Node has a right subtree.** The successor is the **leftmost node** in the right subtree (the minimum of the right subtree).

```
          [8]
         /   \
       [3]   [10]          Successor of 8?
      / \    /  \           8 has a right subtree.
    [1] [6] [9] [14]       Go right to [10], then leftmost = [9].
       / \                  Successor of 8 is 9.
     [4] [7]
```

**Case 2: Node has no right subtree.** Walk up from the node until you find an ancestor where the node is in the left subtree. That ancestor is the successor.

```
          [8]
         /   \
       [3]   [10]          Successor of 7?
      / \                   7 has no right subtree.
    [1] [6]                 Walk up: 7 is right child of 6.
       / \                  Walk up: 6 is right child of 3.
     [4] [7]                Walk up: 3 is left child of 8.
                            Successor of 7 is 8.
```

### Code (using parent pointers or iterative from root)

```cpp
// When you only have the root (no parent pointers)
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* target) {
    // Case 1: right subtree exists
    if (target->right)
        return findMin(target->right);

    // Case 2: walk from root, track last left turn
    TreeNode* successor = nullptr;
    while (root) {
        if (target->val < root->val) {
            successor = root;       // this node is a candidate
            root = root->left;
        } else if (target->val > root->val) {
            root = root->right;
        } else {
            break;  // found the target node
        }
    }
    return successor;
}
```

## 🔹 Delete

Deletion is the most complex BST operation. There are **three cases**:

### Case 1: Deleting a Leaf Node

Simply remove it. Set the parent's pointer to null.

```
Delete 4:
          [8]                    [8]
         /   \                  /   \
       [3]   [10]     -->    [3]   [10]
      / \                   / \
    [1] [6]               [1] [6]
       /
     [4]  <-- remove
```

### Case 2: Deleting a Node with One Child

Replace the node with its only child. The child takes the deleted node's position.

```
Delete 10:
          [8]                    [8]
         /   \                  /   \
       [3]   [10]     -->    [3]   [14]
      / \       \           / \
    [1] [6]    [14]       [1] [6]

[10] had one child [14], so [14] replaces [10].
```

### Case 3: Deleting a Node with Two Children

This is the tricky case. You cannot simply remove the node because it has two subtrees. Instead:

1. Find the node's **in-order successor** (smallest value in right subtree) OR **in-order predecessor** (largest value in left subtree).
2. **Copy** the successor's/predecessor's value into the node being deleted.
3. **Delete** the successor/predecessor (which will be Case 1 or Case 2, since the in-order successor of a node with two children has at most one child).

```
Delete 3 (using in-order successor):

Step 1: Find in-order successor of 3.
        Go right to [6], then leftmost = [4].
        Successor is [4].

          [8]
         /   \
       [3]   [10]
      / \       \
    [1] [6]    [14]
       /
     [4]

Step 2: Copy successor's value into [3].

          [8]
         /   \
       [4]   [10]      <-- was [3], now holds 4
      / \       \
    [1] [6]    [14]
       /
     [4]        <-- now a duplicate, must delete

Step 3: Delete the original [4] (leaf — Case 1).

          [8]
         /   \
       [4]   [10]
      / \       \
    [1] [6]    [14]

BST property is preserved!
```

### Code

```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;

    // Navigate to the node
    if (key < root->val) {
        root->left = deleteNode(root->left, key);
    } else if (key > root->val) {
        root->right = deleteNode(root->right, key);
    } else {
        // Found the node to delete

        // Case 1 & 2: zero or one child
        if (!root->left) {
            TreeNode* temp = root->right;
            delete root;
            return temp;
        }
        if (!root->right) {
            TreeNode* temp = root->left;
            delete root;
            return temp;
        }

        // Case 3: two children
        // Find in-order successor (min of right subtree)
        TreeNode* successor = findMin(root->right);

        // Copy successor's value
        root->val = successor->val;

        // Delete the successor from right subtree
        root->right = deleteNode(root->right, successor->val);
    }
    return root;
}
```

## 🔹 In-Order Traversal Gives Sorted Output

A fundamental BST property: an **in-order traversal** (left, node, right) visits all nodes in **ascending sorted order**.

```
          [8]
         /   \
       [3]   [10]
      / \       \
    [1] [6]    [14]
       / \
     [4] [7]

In-order: 1, 3, 4, 6, 7, 8, 10, 14  --> sorted!
```

This is extremely useful. If you ever need sorted output from a BST, just do an in-order traversal in O(n). See [[Tree Traversals]] for implementation details.

## 🔹 Operations Complexity Summary

| Operation | Average Case | Worst Case (skewed) |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Find Min/Max | O(log n) | O(n) |
| In-order traversal | O(n) | O(n) |
| Space (tree storage) | O(n) | O(n) |

The worst case occurs when the tree is **degenerate/skewed** (essentially a linked list). This is why [[Balanced BST]] variants (AVL, Red-Black) exist — they guarantee O(log n) height.

## 🔹 When to Use a BST

- You need a **dynamic sorted collection** with efficient search, insert, and delete.
- You need to frequently find **min/max**, **successor/predecessor**, or **range queries**.
- You want **sorted iteration** without the cost of sorting.
- In practice, use `std::set`, `std::map` (C++), `TreeSet`, `TreeMap` (Java) — these are implemented as Red-Black trees under the hood. See [[Balanced BST]].

## 🔹 Common Pitfalls

1. **Forgetting the worst case.** A BST built from sorted input degrades to a linked list. Inserting [1, 2, 3, 4, 5] in order produces a right-skewed tree with O(n) operations. Always mention this when discussing BST complexity.

2. **BST property is about the entire subtree, not just children.** A common mistake: checking only that `left->val < node->val < right->val`. You must ensure ALL nodes in the left subtree are less, and ALL nodes in the right subtree are greater.

3. **Delete with two children — choosing successor vs predecessor.** Both work. Just be consistent. Most textbooks use the in-order successor (min of right subtree). The key insight is that the successor has at most one child, so deleting it is always Case 1 or Case 2.

4. **Not updating the parent's pointer during deletion.** The recursive approach handles this naturally by returning the (possibly changed) subtree root. The iterative approach requires explicit parent tracking.

5. **Handling duplicates.** Standard BSTs do not allow duplicates. If duplicates are needed, decide on a convention: store a count at each node, or consistently place duplicates in the left or right subtree (and document it).

## 🔹 Validating a BST

A classic interview question: given a binary tree, determine if it is a valid BST.

The trick is to track the **valid range** for each node, not just compare with the parent.

```cpp
bool isValidBST(TreeNode* node, long minVal = LONG_MIN, long maxVal = LONG_MAX) {
    if (!node) return true;

    if (node->val <= minVal || node->val >= maxVal)
        return false;

    return isValidBST(node->left, minVal, node->val) &&
           isValidBST(node->right, node->val, maxVal);
}
```

Alternatively, do an in-order traversal and check that the output is strictly increasing.
