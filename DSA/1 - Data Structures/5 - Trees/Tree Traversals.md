---
tags:
  - algorithms
  - data-structure
  - tree
  - traversal
---

## 🔹 Real-World Analogy

Imagine you are exploring a building with rooms connected by hallways (a tree). There are different strategies for visiting every room:

- **DFS (Depth-First Search):** Go as deep as possible down one hallway before backtracking. Like exploring a maze by always taking the first turn until you hit a dead end, then backtracking.
- **BFS (Breadth-First Search):** Visit all rooms on the current floor before going to the next floor. Like a fire spreading level by level.

DFS has three variants depending on **when** you process the current room relative to its sub-rooms: before (pre-order), between (in-order), or after (post-order).

## 🔹 Overview

There are four fundamental tree traversal methods. All visit every node exactly once in O(n) time.

| Traversal | Order | Common Use Cases |
|---|---|---|
| **In-order** | Left, Node, Right | Sorted output from BST |
| **Pre-order** | Node, Left, Right | Serialize/copy a tree, prefix expression |
| **Post-order** | Left, Right, Node | Delete tree, postfix expression, bottom-up calc |
| **Level-order** | Level by level, left to right | BFS, shortest path in unweighted tree, level-by-level processing |

## 🔹 Reference Tree

All examples below use this tree:

```
            [4]
           /   \
         [2]   [6]
        / \    / \
      [1] [3][5] [7]
```

| Traversal | Visit Order |
|---|---|
| In-order | 1, 2, 3, 4, 5, 6, 7 |
| Pre-order | 4, 2, 1, 3, 6, 5, 7 |
| Post-order | 1, 3, 2, 5, 7, 6, 4 |
| Level-order | 4, 2, 6, 1, 3, 5, 7 |

---

## 🔹 In-Order Traversal (Left, Node, Right)

Visit the left subtree, then the current node, then the right subtree. For a [[Binary Search Tree]], this produces values in **ascending sorted order**.

### Visualization

```
            [4]
           /   \
         [2]   [6]
        / \    / \
      [1] [3][5] [7]

Traversal path:
  Go left to [2]
    Go left to [1]
      No left child
      VISIT [1]        --> 1
      No right child
    VISIT [2]           --> 2
    Go right to [3]
      No left child
      VISIT [3]         --> 3
      No right child
  VISIT [4]             --> 4
  Go right to [6]
    Go left to [5]
      No left child
      VISIT [5]         --> 5
      No right child
    VISIT [6]           --> 6
    Go right to [7]
      No left child
      VISIT [7]         --> 7
      No right child

Result: 1, 2, 3, 4, 5, 6, 7
```

### Recursive Implementation

```cpp
void inorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    inorder(root->left, result);     // L
    result.push_back(root->val);     // N
    inorder(root->right, result);    // R
}
```

### Iterative Implementation

The iterative version simulates the call stack manually. The key idea: keep going left and pushing nodes onto the stack. When you cannot go left anymore, pop a node, visit it, then go to its right child.

```cpp
vector<int> inorderIterative(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> stk;
    TreeNode* curr = root;

    while (curr || !stk.empty()) {
        // Push all left children onto stack
        while (curr) {
            stk.push(curr);
            curr = curr->left;
        }

        // Pop and visit
        curr = stk.top();
        stk.pop();
        result.push_back(curr->val);   // VISIT

        // Move to right subtree
        curr = curr->right;
    }

    return result;
}
```

**Stack trace for reference tree:**

```
curr=[4], push 4, go left
curr=[2], push 2, go left
curr=[1], push 1, go left
curr=null, stop inner loop

Stack: [4, 2, 1]  --> pop 1, visit 1, go right (null)
Stack: [4, 2]     --> pop 2, visit 2, go right to [3]
curr=[3], push 3, go left (null)
Stack: [4, 3]     --> pop 3, visit 3, go right (null)
Stack: [4]        --> pop 4, visit 4, go right to [6]
curr=[6], push 6, go left
curr=[5], push 5, go left (null)
Stack: [6, 5]     --> pop 5, visit 5, go right (null)
Stack: [6]        --> pop 6, visit 6, go right to [7]
curr=[7], push 7, go left (null)
Stack: [7]        --> pop 7, visit 7, go right (null)
Stack: empty, curr=null --> done

Result: 1, 2, 3, 4, 5, 6, 7
```

### When to Use In-Order

- Get **sorted output** from a BST.
- Validate that a binary tree is a valid BST (check that in-order output is strictly increasing).
- Find the kth smallest element in a BST (do in-order traversal and stop at the kth visit).

---

## 🔹 Pre-Order Traversal (Node, Left, Right)

Visit the current node **first**, then left subtree, then right subtree. The root is always the first element visited.

### Visualization

```
            [4]
           /   \
         [2]   [6]
        / \    / \
      [1] [3][5] [7]

Traversal path:
  VISIT [4]             --> 4
  Go left to [2]
    VISIT [2]           --> 2
    Go left to [1]
      VISIT [1]         --> 1
      No children
    Go right to [3]
      VISIT [3]         --> 3
      No children
  Go right to [6]
    VISIT [6]           --> 6
    Go left to [5]
      VISIT [5]         --> 5
      No children
    Go right to [7]
      VISIT [7]         --> 7
      No children

Result: 4, 2, 1, 3, 6, 5, 7
```

### Recursive Implementation

```cpp
void preorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    result.push_back(root->val);     // N
    preorder(root->left, result);    // L
    preorder(root->right, result);   // R
}
```

### Iterative Implementation

Push the current node, visit it, then push right child first (so left is processed first from the stack — LIFO).

```cpp
vector<int> preorderIterative(TreeNode* root) {
    vector<int> result;
    if (!root) return result;

    stack<TreeNode*> stk;
    stk.push(root);

    while (!stk.empty()) {
        TreeNode* node = stk.top();
        stk.pop();
        result.push_back(node->val);    // VISIT

        // Push RIGHT first so LEFT is processed first (LIFO)
        if (node->right) stk.push(node->right);
        if (node->left)  stk.push(node->left);
    }

    return result;
}
```

**Stack trace:**

```
Push [4]
Pop [4], visit 4, push right [6], push left [2]
Stack: [6, 2]

Pop [2], visit 2, push right [3], push left [1]
Stack: [6, 3, 1]

Pop [1], visit 1, no children
Stack: [6, 3]

Pop [3], visit 3, no children
Stack: [6]

Pop [6], visit 6, push right [7], push left [5]
Stack: [7, 5]

Pop [5], visit 5, no children
Stack: [7]

Pop [7], visit 7, no children
Stack: empty --> done

Result: 4, 2, 1, 3, 6, 5, 7
```

### When to Use Pre-Order

- **Serialize / deserialize** a tree (save and reconstruct). Pre-order captures the structure naturally because the root comes first.
- **Copy / clone** a tree.
- Create a **prefix expression** from an expression tree.
- Whenever you need to process parents before children.

---

## 🔹 Post-Order Traversal (Left, Right, Node)

Visit the left subtree, then the right subtree, then the current node **last**. The root is always the last element visited.

### Visualization

```
            [4]
           /   \
         [2]   [6]
        / \    / \
      [1] [3][5] [7]

Traversal path:
  Go left to [2]
    Go left to [1]
      No children
      VISIT [1]         --> 1
    Go right to [3]
      No children
      VISIT [3]         --> 3
    VISIT [2]           --> 2
  Go right to [6]
    Go left to [5]
      No children
      VISIT [5]         --> 5
    Go right to [7]
      No children
      VISIT [7]         --> 7
    VISIT [6]           --> 6
  VISIT [4]             --> 4

Result: 1, 3, 2, 5, 7, 6, 4
```

### Recursive Implementation

```cpp
void postorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    postorder(root->left, result);   // L
    postorder(root->right, result);  // R
    result.push_back(root->val);     // N
}
```

### Iterative Implementation

Post-order iterative is the trickiest of the three DFS traversals. The challenge is that you need to visit a node **after** both its children, but when you first encounter it via the stack you have not visited the right child yet.

**Method: Two-stack approach.** Pre-order is Node-Left-Right. If we do Node-Right-Left (modified pre-order) and then **reverse** the result, we get Left-Right-Node (post-order).

```cpp
vector<int> postorderIterative(TreeNode* root) {
    vector<int> result;
    if (!root) return result;

    stack<TreeNode*> stk;
    stk.push(root);

    while (!stk.empty()) {
        TreeNode* node = stk.top();
        stk.pop();
        result.push_back(node->val);

        // Push LEFT first, then RIGHT (opposite of pre-order)
        // This gives Node-Right-Left order
        if (node->left)  stk.push(node->left);
        if (node->right) stk.push(node->right);
    }

    // Reverse to get Left-Right-Node
    reverse(result.begin(), result.end());
    return result;
}
```

**Alternative: Single-stack with a `prev` pointer** (no reversal needed).

This approach tracks the previously visited node. When the top of the stack has no children, or its children have already been visited (tracked via `prev`), it is safe to visit and pop it.

```cpp
vector<int> postorderIterativeSingleStack(TreeNode* root) {
    vector<int> result;
    if (!root) return result;

    stack<TreeNode*> stk;
    stk.push(root);
    TreeNode* prev = nullptr;

    while (!stk.empty()) {
        TreeNode* curr = stk.top();

        // Going down: if curr has unvisited children, push them
        if (!prev || prev->left == curr || prev->right == curr) {
            // Coming from parent — try to go deeper
            if (curr->left)
                stk.push(curr->left);
            else if (curr->right)
                stk.push(curr->right);
        }
        // Coming up from the left child
        else if (curr->left == prev) {
            if (curr->right)
                stk.push(curr->right);
        }
        // Coming up from the right child (or no children left)
        else {
            result.push_back(curr->val);   // VISIT
            stk.pop();
        }

        prev = curr;
    }

    return result;
}
```

### When to Use Post-Order

- **Delete / free a tree** (must delete children before parent).
- **Bottom-up calculations**: computing the height or size of a tree, evaluating expression trees.
- Any problem where you need information from **both subtrees** before processing the current node (e.g., checking if a tree is balanced — need heights of left and right first).
- Create a **postfix expression** from an expression tree.

---

## 🔹 Level-Order Traversal (BFS)

Visit all nodes level by level, from left to right. Uses a **queue** (FIFO) instead of a stack.

### Visualization

```
            [4]          Level 0:  [4]
           /   \
         [2]   [6]       Level 1:  [2] [6]
        / \    / \
      [1] [3][5] [7]     Level 2:  [1] [3] [5] [7]

Result: 4, 2, 6, 1, 3, 5, 7
```

### Implementation

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int levelSize = q.size();   // Number of nodes at current level
        vector<int> currentLevel;

        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front();
            q.pop();
            currentLevel.push_back(node->val);

            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }

        result.push_back(currentLevel);
    }

    return result;
}
```

**Queue trace:**

```
Queue: [4]
Level 0: size=1
  Dequeue [4], visit 4, enqueue [2],[6]
  Level result: [4]

Queue: [2, 6]
Level 1: size=2
  Dequeue [2], visit 2, enqueue [1],[3]
  Dequeue [6], visit 6, enqueue [5],[7]
  Level result: [2, 6]

Queue: [1, 3, 5, 7]
Level 2: size=4
  Dequeue [1], visit 1, no children
  Dequeue [3], visit 3, no children
  Dequeue [5], visit 5, no children
  Dequeue [7], visit 7, no children
  Level result: [1, 3, 5, 7]

Queue: empty --> done

Result: [[4], [2, 6], [1, 3, 5, 7]]
```

The key technique is capturing `q.size()` at the start of each level. This tells you how many nodes belong to the current level before any children from the next level are enqueued.

### Flat Version (No Level Grouping)

If you just need a flat list and do not need to distinguish levels:

```cpp
vector<int> levelOrderFlat(TreeNode* root) {
    vector<int> result;
    if (!root) return result;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        TreeNode* node = q.front();
        q.pop();
        result.push_back(node->val);

        if (node->left)  q.push(node->left);
        if (node->right) q.push(node->right);
    }

    return result;
}
```

### When to Use Level-Order

- **Level-by-level processing**: print each level, find the maximum/minimum value at each level, right-side view of a tree.
- **Shortest path** in an unweighted tree (BFS finds shortest path by number of edges).
- **Connect nodes at the same level** (next-right-pointer problems).
- **Zigzag traversal** (alternate left-to-right and right-to-left at each level).
- Whenever you need to process nodes by their **distance from the root**.

---

## 🔹 DFS vs BFS Comparison

| Aspect | DFS (In/Pre/Post) | BFS (Level-Order) |
|---|---|---|
| Data structure | Stack (or recursion) | Queue |
| Space complexity | O(h) where h = height | O(w) where w = max width |
| Best for deep trees | Yes (less memory) | No (wide levels eat memory) |
| Best for wide trees | No (deep recursion) | Yes (levels are small) |
| Finds shortest path? | No | Yes (in unweighted tree) |
| Implementation | Recursive is simple | Iterative with queue |

Space complexity notes:
- **DFS**: O(h) = O(log n) for balanced trees, O(n) for skewed trees.
- **BFS**: O(w) = O(n) for the last level of a complete tree (up to n/2 nodes), O(1) for skewed trees.

## 🔹 When to Use Which Traversal — Decision Guide

```
Need sorted output from BST?
  --> In-order

Need to serialize / copy a tree?
  --> Pre-order

Need to delete a tree / bottom-up calculation?
  --> Post-order

Need level-by-level processing?
  --> Level-order (BFS)

Need to process parent before children?
  --> Pre-order

Need to process children before parent?
  --> Post-order

Need shortest path by edges?
  --> Level-order (BFS)

Just need to visit all nodes?
  --> Any traversal works (pick the simplest for the problem)
```

## 🔹 Morris Traversal (Bonus — O(1) Space)

Morris traversal achieves in-order traversal in O(n) time and **O(1) space** (no stack, no recursion) by temporarily modifying the tree using **threaded binary tree** links.

The idea: before going left, create a temporary link from the in-order predecessor back to the current node. This link replaces the stack — when you reach a null at the bottom of the left subtree, the thread takes you back up.

```cpp
vector<int> morrisInorder(TreeNode* root) {
    vector<int> result;
    TreeNode* curr = root;

    while (curr) {
        if (!curr->left) {
            // No left subtree: visit and go right
            result.push_back(curr->val);
            curr = curr->right;
        } else {
            // Find in-order predecessor (rightmost in left subtree)
            TreeNode* pred = curr->left;
            while (pred->right && pred->right != curr)
                pred = pred->right;

            if (!pred->right) {
                // First visit: create thread and go left
                pred->right = curr;
                curr = curr->left;
            } else {
                // Second visit: remove thread, visit, go right
                pred->right = nullptr;
                result.push_back(curr->val);
                curr = curr->right;
            }
        }
    }

    return result;
}
```

Morris traversal is rarely required in interviews but impressive to know. The key trade-off: O(1) space at the cost of temporarily modifying the tree (and added complexity).

## 🔹 Common Pitfalls

1. **Confusing the three DFS orderings.** Remember: the name tells you when the **N**ode is visited relative to its children. Pre = before, In = in between, Post = after.

2. **Iterative post-order is hard.** If you need iterative post-order in an interview, use the "modified pre-order + reverse" trick. It is much simpler and less error-prone.

3. **Forgetting `levelSize` in BFS.** Without capturing `q.size()` at the start of each level, you cannot distinguish which nodes belong to which level. This is critical for problems that ask for per-level results.

4. **Stack overflow with recursive DFS on deep trees.** For trees that can be very deep (up to 10^5 nodes in a skewed tree), use the iterative version to avoid stack overflow.

5. **Assuming traversal order without checking.** When a problem says "traverse the tree," clarify which traversal. The order matters for the correctness of many algorithms.

## 🔹 Complexity Summary

| Traversal | Time | Space |
|---|---|---|
| In-order (recursive) | O(n) | O(h) call stack |
| In-order (iterative) | O(n) | O(h) explicit stack |
| Pre-order (recursive) | O(n) | O(h) call stack |
| Pre-order (iterative) | O(n) | O(h) explicit stack |
| Post-order (recursive) | O(n) | O(h) call stack |
| Post-order (iterative) | O(n) | O(h) explicit stack |
| Level-order (BFS) | O(n) | O(w) queue |
| Morris in-order | O(n) | O(1) |

Where `h` = height of tree, `w` = maximum width of tree, `n` = number of nodes.

For a balanced tree: `h = O(log n)`, `w = O(n)`.
For a skewed tree: `h = O(n)`, `w = O(1)`.
