---
tags:
  - algorithms
  - data-structure
  - linked-list
---

## 🔹 Real-World Analogy

Think of a **treasure hunt** (scavenger hunt). You start with a single clue — the **first clue** — and each clue tells you two things: something interesting at the current location (the **data**) and where to find the **next clue** (the pointer). You can only move forward, and the hunt ends when a clue says "THE END" (NULL).

Key insight: you cannot jump directly to clue #5. You **must** follow the chain from clue #1 onward. This is the fundamental trade-off of a linked list vs an [[Arrays|array]].

---

## 🔹 Node Structure

Each element in a singly linked list is a **node** containing two fields:

1. **data** — the value stored
2. **next** — a pointer to the next node in the list (or NULL if this is the last node)

```
         Node
    +----------+------+
    |   data   | next ------>
    +----------+------+
```

```cpp
struct Node {
    int data;
    Node* next;
};
```

---

## 🔹 Head Pointer

The **head** is a pointer to the first node. It is the single entry point into the entire list. If `head == NULL`, the list is empty.

- You do NOT store the list itself — you only store the head pointer.
- Lose the head pointer = lose the entire list (memory leak).

```
  head
   |
   v
  [10|*]-->[20|*]-->[30|*]-->[40|NULL]
```

---

## 🔹 Linked List Visualization

```
  head
   |
   v
  +----+------+    +----+------+    +----+------+    +----+------+
  | 10 | next-|--->| 20 | next-|--->| 30 | next-|--->| 40 | NULL |
  +----+------+    +----+------+    +----+------+    +----+------+
```

Each box is a node. The `next` field holds the memory address of the following node. The last node's `next` is NULL, signaling the end.

---

## 🔹 Operations

### Insert at Head — O(1)

Create a new node, point its `next` to the current head, then update `head`.

```
  BEFORE:
    head --> [20|*]-->[30|*]-->[40|NULL]

  STEP 1: Create new node with data = 10
    newNode --> [10|NULL]

  STEP 2: Point newNode->next to head
    newNode --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]

  STEP 3: Update head = newNode
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]
```

```cpp
void insertAtHead(Node*& head, int val) {
    Node* newNode = new Node{val, head};
    head = newNode;
}
```

**Why O(1)?** No traversal needed. We always know where the head is.

---

### Insert at Tail — O(n)

Must traverse the entire list to find the last node, then attach the new node.

```
  BEFORE:
    head --> [10|*]-->[20|*]-->[30|NULL]

  STEP 1: Create newNode with data = 40
    newNode --> [40|NULL]

  STEP 2: Traverse to the last node (30)
    head --> [10|*]-->[20|*]-->[30|NULL]
                                 ^
                                curr (last node found)

  STEP 3: Set curr->next = newNode
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]
```

```cpp
void insertAtTail(Node*& head, int val) {
    Node* newNode = new Node{val, nullptr};
    if (head == nullptr) {
        head = newNode;
        return;
    }
    Node* curr = head;
    while (curr->next != nullptr) {
        curr = curr->next;
    }
    curr->next = newNode;
}
```

**Optimization:** Maintain a **tail pointer** alongside head. Then insert at tail becomes O(1) — just update `tail->next = newNode` and `tail = newNode`.

---

### Insert at Position k — O(k)

Traverse to node at position k-1 (the predecessor), then rewire.

```
  INSERT 25 at position 2 (0-indexed):

  BEFORE:
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]
              pos 0    pos 1    pos 2    pos 3

  STEP 1: Traverse to position k-1 = 1
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]
                        ^
                       prev (position 1)

  STEP 2: Create newNode, set newNode->next = prev->next
    newNode --> [25|*]-->[30|*]-->[40|NULL]

  STEP 3: Set prev->next = newNode
    head --> [10|*]-->[20|*]-->[25|*]-->[30|*]-->[40|NULL]
```

```cpp
void insertAtPosition(Node*& head, int val, int pos) {
    if (pos == 0) {
        insertAtHead(head, val);
        return;
    }
    Node* prev = head;
    for (int i = 0; i < pos - 1 && prev != nullptr; i++) {
        prev = prev->next;
    }
    if (prev == nullptr) return;  // position out of bounds
    Node* newNode = new Node{val, prev->next};
    prev->next = newNode;
}
```

---

### Delete a Node — O(n)

Find the predecessor of the target node, rewire around it, then free the memory.

```
  DELETE node with value 20:

  BEFORE:
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]

  STEP 1: Find predecessor of node with value 20
    head --> [10|*]-->[20|*]-->[30|*]-->[40|NULL]
              ^        ^
             prev    target

  STEP 2: Set prev->next = target->next
    head --> [10|*]---------->[30|*]-->[40|NULL]
                      [20|*] (orphaned, to be freed)

  STEP 3: Free target
    head --> [10|*]-->[30|*]-->[40|NULL]
```

Special case: deleting the **head** node requires updating the head pointer itself.

```cpp
void deleteNode(Node*& head, int val) {
    if (head == nullptr) return;

    // Special case: deleting head
    if (head->data == val) {
        Node* temp = head;
        head = head->next;
        delete temp;
        return;
    }

    Node* prev = head;
    while (prev->next != nullptr && prev->next->data != val) {
        prev = prev->next;
    }
    if (prev->next == nullptr) return;  // not found

    Node* target = prev->next;
    prev->next = target->next;
    delete target;
}
```

---

### Search — O(n)

Linear scan from head until the value is found or we reach NULL.

```cpp
bool search(Node* head, int val) {
    Node* curr = head;
    while (curr != nullptr) {
        if (curr->data == val) return true;
        curr = curr->next;
    }
    return false;
}
```

---

### Reverse the List — O(n)

This is one of the most important linked list algorithms. The idea: walk through the list and reverse each pointer to point **backward** instead of forward. Track three pointers: `prev`, `curr`, `next`.

```
  BEFORE:
    NULL    [10|*]-->[20|*]-->[30|*]-->[40|NULL]
     ^       ^
    prev    curr

  ITERATION 1:
    - next = curr->next (save 20)
    - curr->next = prev  (10 now points to NULL)
    - prev = curr        (prev moves to 10)
    - curr = next        (curr moves to 20)

    NULL<--[10|NULL]    [20|*]-->[30|*]-->[40|NULL]
             ^           ^
            prev        curr

  ITERATION 2:
    - next = curr->next (save 30)
    - curr->next = prev  (20 now points to 10)
    - prev = curr        (prev moves to 20)
    - curr = next        (curr moves to 30)

    NULL<--[10|NULL]<--[20|*]    [30|*]-->[40|NULL]
                         ^        ^
                        prev    curr

  ITERATION 3:
    - next = curr->next (save 40)
    - curr->next = prev  (30 now points to 20)
    - prev = curr        (prev moves to 30)
    - curr = next        (curr moves to 40)

    NULL<--[10]<--[20]<--[30|*]    [40|NULL]
                           ^        ^
                          prev    curr

  ITERATION 4:
    - next = curr->next (save NULL)
    - curr->next = prev  (40 now points to 30)
    - prev = curr        (prev moves to 40)
    - curr = next        (curr moves to NULL)

    NULL<--[10]<--[20]<--[30]<--[40|*]    NULL
                                  ^        ^
                                 prev    curr

  curr == NULL, loop ends.
  Set head = prev.

  AFTER:
    head --> [40|*]-->[30|*]-->[20|*]-->[10|NULL]
```

```cpp
Node* reverseList(Node* head) {
    Node* prev = nullptr;
    Node* curr = head;
    while (curr != nullptr) {
        Node* next = curr->next;  // save next
        curr->next = prev;        // reverse the link
        prev = curr;              // advance prev
        curr = next;              // advance curr
    }
    return prev;  // prev is the new head
}
```

**Memorize this pattern.** It shows up constantly in interviews — both on its own and as a building block inside harder problems (reverse a sub-section, reverse in k-groups, palindrome check, etc.).

---

### Find Length — O(n)

```cpp
int length(Node* head) {
    int count = 0;
    Node* curr = head;
    while (curr != nullptr) {
        count++;
        curr = curr->next;
    }
    return count;
}
```

---

## 🔹 Slow/Fast Pointer Technique (Floyd's Algorithm)

This is one of the most powerful linked list techniques. Two pointers start at the head:

- **slow** moves 1 step at a time
- **fast** moves 2 steps at a time

### Cycle Detection

If there is a cycle, fast will eventually "lap" slow and they will meet. If there is no cycle, fast reaches NULL.

```
  List with a cycle:

    [1]-->[2]-->[3]-->[4]-->[5]
                 ^              |
                 |              |
                 +--------------+
                 (5 points back to 3)

  Step 0:  S=1, F=1
  Step 1:  S=2, F=3        (slow +1, fast +2)
  Step 2:  S=3, F=5        (slow +1, fast +2)
  Step 3:  S=4, F=4        (slow +1, fast +2 -> 3 -> 4)
           ^--- THEY MEET! Cycle detected.
```

**Why does this work?** Once both pointers are inside the cycle, the fast pointer closes the gap by 1 node per step. They are guaranteed to meet within one full loop of the cycle.

```cpp
bool hasCycle(Node* head) {
    Node* slow = head;
    Node* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

---

### Find the Middle Node

When fast reaches the end (NULL), slow is at the middle.

```
  List: [1]-->[2]-->[3]-->[4]-->[5]-->NULL

  Step 0:  S=1, F=1
  Step 1:  S=2, F=3
  Step 2:  S=3, F=5
           fast->next is NULL, stop.
           slow = 3  <-- MIDDLE

  Even-length list: [1]-->[2]-->[3]-->[4]-->NULL

  Step 0:  S=1, F=1
  Step 1:  S=2, F=3
  Step 2:  S=3, F=NULL (fast == NULL, stop)
           slow = 3  <-- second middle node (upper middle)

  (For "lower middle" on even-length lists, check fast->next->next instead)
```

```
  Visual — pointers moving through a 7-node list:

     S                             F
     v                             v
    [1]-->[2]-->[3]-->[4]-->[5]-->[6]-->[7]-->NULL

               S                             F
               v                             v
    [1]-->[2]-->[3]-->[4]-->[5]-->[6]-->[7]-->NULL

                         S                         F=NULL
                         v
    [1]-->[2]-->[3]-->[4]-->[5]-->[6]-->[7]-->NULL
                         ^
                       MIDDLE
```

```cpp
Node* findMiddle(Node* head) {
    Node* slow = head;
    Node* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

---

## 🔹 Arrays vs Linked Lists

| Operation            | [[Arrays\|Array]] | Singly Linked List |
| -------------------- | ------------------ | ------------------ |
| Access by index      | O(1)               | O(n)               |
| Insert at head       | O(n)               | **O(1)**           |
| Insert at tail       | O(1) amortized*    | O(n) or O(1)**     |
| Insert at middle     | O(n)               | O(n)***            |
| Delete at head       | O(n)               | **O(1)**           |
| Delete at middle     | O(n)               | O(n)               |
| Search (unsorted)    | O(n)               | O(n)               |
| Search (sorted)      | **O(log n)**       | O(n)               |
| Memory per element   | Just the data      | Data + pointer     |
| Cache locality       | **Excellent**      | Poor               |
| Dynamic sizing       | Resize needed      | **Always dynamic** |

\* Dynamic arrays (e.g., `std::vector`) amortize tail insertion to O(1).
\** O(1) if you maintain a tail pointer, O(n) otherwise.
\*** O(n) to find the position, but the actual insertion (pointer rewiring) is O(1).

**Key takeaway:** Arrays win on random access and cache performance. Linked lists win on insertions/deletions at the head and when you need frequent structural changes without shifting elements.

---

## 🔹 Time Complexity Summary

| Operation            | Time     | Notes                              |
| -------------------- | -------- | ---------------------------------- |
| Insert at head       | O(1)     | Best use case for linked lists     |
| Insert at tail       | O(n)     | O(1) with tail pointer             |
| Insert at position k | O(k)     | Traverse to predecessor            |
| Delete (by value)    | O(n)     | Must find predecessor              |
| Delete at head       | O(1)     |                                    |
| Search               | O(n)     | No binary search possible          |
| Access by index      | O(n)     | Must traverse from head            |
| Find length          | O(n)     | O(1) if you cache the count        |
| Reverse              | O(n)     | Single pass, O(1) extra space      |
| Detect cycle         | O(n)     | Floyd's slow/fast pointer          |
| Find middle          | O(n)     | Floyd's slow/fast pointer          |
| Space complexity     | O(n)     | n nodes, each with data + pointer  |

---

## 🔹 Common Pitfalls

1. **Losing reference to the head** — If you overwrite `head` without saving it, the entire list is leaked. Always use a separate traversal pointer (`curr`), never walk `head` itself through the list.

2. **Not handling empty list (head == NULL)** — Every function should check `if (head == nullptr)` before dereferencing. Forgetting this causes segfaults.

3. **Not handling single-node list** — Deleting the only node means setting `head = NULL`. Reversing a single-node list should return it unchanged. Many bugs hide here.

4. **Forgetting to update head on insert/delete at head** — The head pointer must be passed by reference (`Node*&`) or returned. Otherwise changes to head inside a function are lost.

5. **Off-by-one when traversing** — When inserting at position k, you need to stop at position k-1 (the predecessor). A common mistake is stopping one node too early or too late.

6. **Memory leaks on delete** — After rewiring pointers around a deleted node, you must `delete` (or `free`) the removed node. In languages with garbage collection (Java, Python, C#), this is handled automatically.

7. **Not saving `next` before reversing** — In the reversal algorithm, you must save `curr->next` before overwriting it. Otherwise you lose the rest of the list.

---

## 🔹 Template Code

### Node Definition

```cpp
struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};
```

### Insert at Head

```cpp
void insertAtHead(Node*& head, int val) {
    Node* newNode = new Node(val);
    newNode->next = head;
    head = newNode;
}
```

### Delete a Node by Value

```cpp
void deleteNode(Node*& head, int val) {
    if (!head) return;

    if (head->data == val) {
        Node* temp = head;
        head = head->next;
        delete temp;
        return;
    }

    Node* prev = head;
    while (prev->next && prev->next->data != val) {
        prev = prev->next;
    }

    if (!prev->next) return;

    Node* target = prev->next;
    prev->next = target->next;
    delete target;
}
```

### Reverse a Linked List (Iterative)

```cpp
Node* reverseList(Node* head) {
    Node* prev = nullptr;
    Node* curr = head;

    while (curr) {
        Node* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }

    return prev;  // new head
}
```

### Detect Cycle (Floyd's)

```cpp
bool hasCycle(Node* head) {
    Node* slow = head;
    Node* fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }

    return false;
}
```

### Find Middle Node

```cpp
Node* findMiddle(Node* head) {
    Node* slow = head;
    Node* fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;  // middle node
}
```

---

## 🔹 When to Use Linked Lists vs Arrays

**Use a linked list when:**
- You need frequent insertions/deletions at the beginning
- You don't know the size in advance and want true dynamic sizing without reallocation
- You are implementing a [[Stack]] (LIFO — push/pop at head, both O(1))
- You are implementing a [[Queue]] (FIFO — with head and tail pointers)
- You need to merge or split sequences efficiently
- The problem explicitly gives you nodes with `next` pointers (LeetCode-style)

**Use an array when:**
- You need random access by index
- You need cache-friendly traversal (arrays are stored contiguously in memory)
- You need binary search on sorted data
- Memory overhead per element matters (no extra pointer storage)
- You mostly append at the end (dynamic arrays handle this well)

---

## 🔹 Related Concepts

- [[Doubly Linked List]] — each node has both `next` and `prev` pointers, enabling O(1) delete if you have a reference to the node
- [[Arrays]] — the primary alternative; compare trade-offs above
- [[Stack]] — can be implemented with a singly linked list (push/pop at head)
- [[Queue]] — can be implemented with head + tail pointers on a singly linked list
- [[Hash Map and Hash Set]] — often use linked lists internally for chaining (collision resolution)
- [[Recursion]] — many linked list problems have elegant recursive solutions (reverse, merge, etc.)
