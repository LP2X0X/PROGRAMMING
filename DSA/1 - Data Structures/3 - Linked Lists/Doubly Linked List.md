---
tags:
  - algorithms
  - data-structure
  - linked-list
---

## 🔹 Real-World Analogy

Think of a **music playlist** with previous/next buttons. Each song knows:
- What song comes **after** it (next track)
- What song comes **before** it (previous track)

You can skip forward or backward from any point. If you want to remove a song, you just reconnect the song before it to the song after it -- no need to start from the beginning of the playlist.

Another analogy: an **undo/redo** history. Each action links to the previous action (undo) and the next action (redo), allowing bidirectional navigation through your edit history.

---

## 🔹 Node Structure

Each node carries three pieces of information:

```
+--------+--------+--------+
|  prev  |  data  |  next  |
+--------+--------+--------+
    |                  |
    v                  v
 pointer to         pointer to
 previous node      next node
```

```cpp
struct Node {
    int data;
    Node* prev;
    Node* next;
};
```

Compared to a [[Singly Linked List]] node (which only has `data` + `next`), this uses one extra pointer per node. That extra pointer is what buys us bidirectional traversal and O(1) deletion.

---

## 🔹 Structure of a Doubly Linked List

```
        head                                           tail
         |                                              |
         v                                              v
NULL <-- [prev|A|next] <--> [prev|B|next] <--> [prev|C|next] --> NULL
```

Expanded view showing each pointer:

```
         +------+---+------+     +------+---+------+     +------+---+------+
NULL <-- | prev | A | next | <-> | prev | B | next | <-> | prev | C | next | --> NULL
         +------+---+------+     +------+---+------+     +------+---+------+

              head->prev = NULL
              head->next = B
              B->prev    = head (A)
              B->next    = C
              tail->next = NULL
              tail->prev = B
```

Key invariants:
- `head->prev == NULL` (nothing before the first node)
- `tail->next == NULL` (nothing after the last node)
- For any adjacent nodes X and Y: `X->next == Y` implies `Y->prev == X`

---

## 🔹 Advantages Over Singly Linked List

| Capability | [[Singly Linked List]] | Doubly Linked List |
|---|---|---|
| Traverse forward | Yes | Yes |
| Traverse backward | No | Yes |
| Delete node given pointer to it | O(n) -- must find predecessor | **O(1)** -- prev pointer available |
| Insert before a given node | O(n) -- must find predecessor | **O(1)** -- prev pointer available |
| Insert after a given node | O(1) | O(1) |
| Implement [[Deque]] | Awkward (no O(1) remove-from-tail) | Natural fit |
| Memory per node | 1 pointer + data | 2 pointers + data |

The **critical advantage** is O(1) deletion given a reference to the node. This is the foundation of the [[LRU Cache]] design pattern.

---

## 🔹 Operations

### Insert at Head -- O(1)

Insert node X before the current head:

```
BEFORE:
   head
    |
    v
   [prev|A|next] <--> [prev|B|next] --> NULL

AFTER:
   head
    |
    v
   [prev|X|next] <--> [prev|A|next] <--> [prev|B|next] --> NULL
```

```cpp
void insertAtHead(Node*& head, Node*& tail, int val) {
    Node* newNode = new Node{val, nullptr, head};
    if (head != nullptr) {
        head->prev = newNode;
    } else {
        tail = newNode;   // list was empty
    }
    head = newNode;
}
```

### Insert at Tail -- O(1) (with tail pointer)

```
BEFORE:
                                     tail
                                      |
                                      v
   [prev|A|next] <--> [prev|B|next]

AFTER:
                                                        tail
                                                         |
                                                         v
   [prev|A|next] <--> [prev|B|next] <--> [prev|X|next]
```

```cpp
void insertAtTail(Node*& head, Node*& tail, int val) {
    Node* newNode = new Node{val, tail, nullptr};
    if (tail != nullptr) {
        tail->next = newNode;
    } else {
        head = newNode;   // list was empty
    }
    tail = newNode;
}
```

### Insert After a Given Node -- O(1)

Insert X after node A, where A->next currently points to B:

```
BEFORE:
   ... <--> [prev|A|next] <--> [prev|B|next] <--> ...

Step 1: X->prev = A, X->next = B
   ... <--> [prev|A|next] <--> [prev|B|next] <--> ...
                    ^               ^
                    |               |
                [prev|X|next]

Step 2: B->prev = X
Step 3: A->next = X

AFTER:
   ... <--> [prev|A|next] <--> [prev|X|next] <--> [prev|B|next] <--> ...
```

```cpp
void insertAfter(Node* target, Node*& tail, int val) {
    Node* newNode = new Node{val, target, target->next};
    if (target->next != nullptr) {
        target->next->prev = newNode;
    } else {
        tail = newNode;   // target was the tail
    }
    target->next = newNode;
}
```

### Delete a Given Node -- O(1) (The Key Advantage)

This is where doubly linked lists shine. Given a direct pointer to node X, remove it by connecting its neighbors to each other:

```
BEFORE:
   ... <--> [prev|A|next] <--> [prev|X|next] <--> [prev|B|next] <--> ...

Step 1: A->next = B       (skip over X going forward)
Step 2: B->prev = A       (skip over X going backward)

AFTER:
   ... <--> [prev|A|next] <--> [prev|B|next] <--> ...
                                                        X is now orphaned
```

Why this is O(n) in a singly linked list: to delete X you need to update `predecessor->next`, but a singly linked list node does not know its predecessor. You must traverse from the head to find it. With a doubly linked list, `X->prev` gives you the predecessor directly.

```cpp
void deleteNode(Node* target, Node*& head, Node*& tail) {
    if (target->prev != nullptr) {
        target->prev->next = target->next;
    } else {
        head = target->next;   // target was the head
    }
    if (target->next != nullptr) {
        target->next->prev = target->prev;
    } else {
        tail = target->prev;   // target was the tail
    }
    delete target;
}
```

### Search -- O(n)

No shortcut; must traverse node by node, same as singly linked list.

```cpp
Node* search(Node* head, int val) {
    Node* curr = head;
    while (curr != nullptr) {
        if (curr->data == val) return curr;
        curr = curr->next;
    }
    return nullptr;
}
```

### Reverse -- O(n)

Swap `prev` and `next` for every node, then swap head and tail pointers:

```
BEFORE: NULL <-- [A] <--> [B] <--> [C] --> NULL
                 head                tail

Swap pointers in each node:
  A: swap(prev, next)
  B: swap(prev, next)
  C: swap(prev, next)

AFTER:  NULL <-- [C] <--> [B] <--> [A] --> NULL
                 head                tail
```

```cpp
void reverse(Node*& head, Node*& tail) {
    Node* curr = head;
    while (curr != nullptr) {
        std::swap(curr->prev, curr->next);
        curr = curr->prev;  // prev is now the old next
    }
    std::swap(head, tail);
}
```

---

## 🔹 LRU Cache Design Pattern

This is one of the most important applications of a doubly linked list and a **classic interview question** (LeetCode 146). An LRU (Least Recently Used) Cache evicts the least recently accessed item when the cache is full.

### The Insight: Hash Map + Doubly Linked List

An LRU Cache needs two operations to be fast:
1. **Lookup** a key -- O(1) --> use a [[Hash Map and Hash Set|Hash Map]]
2. **Evict** the least recently used item -- O(1) --> use a Doubly Linked List

The doubly linked list maintains **recency order**: most recently used at the head, least recently used at the tail.

### Why a Doubly Linked List (Not Singly)?

When a cached item is accessed, you must:
1. **Find** the node in O(1) -- the hash map stores a pointer directly to the node
2. **Remove** the node from its current position in O(1) -- requires `prev` pointer
3. **Move** it to the head (most recently used) in O(1) -- head insertion

Step 2 is why you need a **doubly** linked list. With a singly linked list, removal would be O(n).

### ASCII Art of LRU Cache Structure

```
                      Hash Map                    Doubly Linked List
                 +----------------+        (most recent)         (least recent)
                 |  key -> Node*  |         head                     tail
                 |----------------|          |                        |
                 |  "A" -> -------+------->  [A] <--> [C] <--> [B] <--> [D]
                 |  "B" -> -------+------------------->              ^
                 |  "C" -> -------+----------->  ^                   |
                 |  "D" -> -------+------------------------------->  ^
                 +----------------+

  get("B"):
    1. Hash map lookup: O(1) -> get pointer to node [B]
    2. Remove [B] from current position: O(1) -- prev/next pointers
    3. Insert [B] at head: O(1)

  Result:
    [B] <--> [A] <--> [C] <--> [D]
     ^                          ^
     head                      tail (least recent, evict first)
```

### LRU Cache Operations Summary

| Operation | Steps | Time |
|---|---|---|
| `get(key)` | Map lookup, move node to head | O(1) |
| `put(key, val)` | Map lookup; if exists, update + move to head; if new + full, evict tail then insert at head | O(1) |
| Eviction | Remove tail node, delete from map | O(1) |

### LRU Cache Skeleton Code

```cpp
struct CacheNode {
    int key, value;
    CacheNode* prev;
    CacheNode* next;
};

class LRUCache {
    int capacity;
    std::unordered_map<int, CacheNode*> map;
    CacheNode* head;  // dummy head (sentinel)
    CacheNode* tail;  // dummy tail (sentinel)

public:
    LRUCache(int cap) : capacity(cap) {
        head = new CacheNode{0, 0, nullptr, nullptr};
        tail = new CacheNode{0, 0, nullptr, nullptr};
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (map.find(key) == map.end()) return -1;
        CacheNode* node = map[key];
        removeNode(node);
        insertAtHead(node);
        return node->value;
    }

    void put(int key, int value) {
        if (map.find(key) != map.end()) {
            CacheNode* node = map[key];
            node->value = value;
            removeNode(node);
            insertAtHead(node);
        } else {
            if ((int)map.size() == capacity) {
                CacheNode* lru = tail->prev;   // least recently used
                map.erase(lru->key);
                removeNode(lru);
                delete lru;
            }
            CacheNode* newNode = new CacheNode{key, value, nullptr, nullptr};
            map[key] = newNode;
            insertAtHead(newNode);
        }
    }

private:
    // Remove node from its current position -- O(1)
    void removeNode(CacheNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    // Insert node right after the dummy head -- O(1)
    void insertAtHead(CacheNode* node) {
        node->next = head->next;
        node->prev = head;
        head->next->prev = node;
        head->next = node;
    }
};
```

Notice how sentinel nodes eliminate all the `if (head == nullptr)` edge cases. See the section below for more on this technique.

---

## 🔹 Sentinel Nodes (Dummy Head / Dummy Tail)

A powerful technique to simplify doubly linked list code by eliminating null-pointer edge cases.

### The Problem Without Sentinels

Every insert/delete must handle special cases:
- Is the list empty?
- Is the target the head?
- Is the target the tail?
- Is the target the only node?

This leads to branches like:

```cpp
if (target->prev != nullptr) {
    target->prev->next = target->next;
} else {
    head = target->next;
}
```

### The Solution: Sentinel Nodes

Add two dummy nodes that are never removed:

```
WITHOUT sentinels:
  NULL <-- [A] <--> [B] <--> [C] --> NULL

WITH sentinels:
  [DUMMY_HEAD] <--> [A] <--> [B] <--> [C] <--> [DUMMY_TAIL]
       ^                                             ^
  never removed                                 never removed
  head always valid                             tail always valid

EMPTY list with sentinels:
  [DUMMY_HEAD] <--> [DUMMY_TAIL]
```

Now `removeNode` is always safe -- every real node is guaranteed to have a non-null `prev` and `next`:

```cpp
// No edge cases needed!
void removeNode(Node* node) {
    node->prev->next = node->next;
    node->next->prev = node->prev;
}

void insertAfter(Node* target, Node* newNode) {
    newNode->next = target->next;
    newNode->prev = target;
    target->next->prev = newNode;
    target->next = newNode;
}
```

This is the technique used in the LRU Cache implementation above and is widely recommended for interview problems involving doubly linked lists.

---

## 🔹 When to Use Doubly vs Singly Linked List

| Criteria | Singly Linked List | Doubly Linked List |
|---|---|---|
| Memory per node | `data` + 1 pointer | `data` + 2 pointers |
| Traverse forward | O(n) | O(n) |
| Traverse backward | Not possible | O(n) |
| Insert at head | O(1) | O(1) |
| Insert at tail (with tail ptr) | O(1) | O(1) |
| Delete head | O(1) | O(1) |
| Delete tail | O(n) | O(1) |
| Delete given node (by reference) | O(n) | **O(1)** |
| Insert before given node | O(n) | **O(1)** |
| Implementation complexity | Simpler | More complex |

**Use a singly linked list when:**
- You only need forward traversal (e.g., implementing a stack)
- Memory is a concern and you do not need O(1) deletion by reference
- The list is used as a simple chain (hash table chaining, adjacency lists)

**Use a doubly linked list when:**
- You need O(1) deletion given a pointer to the node (LRU Cache, LFU Cache)
- You need bidirectional traversal (browser history, undo/redo)
- You need an efficient [[Deque]] (double-ended queue)
- You are implementing an ordered data structure that requires frequent reordering

---

## 🔹 Time Complexity Summary

| Operation | Time | Notes |
|---|---|---|
| Insert at head | O(1) | |
| Insert at tail | O(1) | Requires tail pointer |
| Insert after/before given node | O(1) | Given a reference to the node |
| Delete given node (by reference) | O(1) | The key advantage over singly linked |
| Delete head | O(1) | |
| Delete tail | O(1) | Requires tail pointer |
| Search by value | O(n) | Must traverse |
| Access by index | O(n) | No random access (unlike [[Arrays]]) |
| Reverse | O(n) | Swap prev/next for every node |

Space complexity: O(n) total, with 2 pointers of overhead per node compared to an [[Arrays|array]].

---

## 🔹 Common Pitfalls

**1. Forgetting to update BOTH prev and next pointers**
Every insertion or deletion must update pointers in both directions. If you set `A->next = C` but forget `C->prev = A`, the list is broken when traversed backward.

**2. Dangling pointers after deletion**
After removing a node, set the deleted node's pointers to `nullptr` before freeing it, or use smart pointers. Accessing a freed node's `prev` or `next` is undefined behavior.

**3. Not handling head/tail updates**
When inserting or deleting the first or last node, you must update the head/tail pointers. Use sentinel nodes to avoid this entirely.

**4. Off-by-one in traversal**
When using sentinel nodes, the first real element is `head->next` (not `head`), and the last is `tail->prev` (not `tail`). Iterating should stop when you hit the sentinel, not when the pointer is null.

**5. Extra memory cost**
Each node uses one extra pointer compared to a singly linked list. For large lists of small data (e.g., a list of integers), this overhead is significant. Consider whether you truly need the doubly linked list's capabilities.

**6. Circular reference issues (in garbage-collected languages)**
In languages with GC (Java, C#, Python), doubly linked lists create circular references between adjacent nodes. Modern GCs handle this, but it is worth being aware of.

---

## 🔹 Template Code

### Node Definition

```cpp
struct DLLNode {
    int data;
    DLLNode* prev;
    DLLNode* next;

    DLLNode(int val) : data(val), prev(nullptr), next(nullptr) {}
};
```

### Insert at Head

```cpp
void insertAtHead(DLLNode*& head, DLLNode*& tail, int val) {
    DLLNode* newNode = new DLLNode(val);
    newNode->next = head;
    if (head) head->prev = newNode;
    else tail = newNode;
    head = newNode;
}
```

### Delete a Given Node -- O(1)

```cpp
void deleteNode(DLLNode* node, DLLNode*& head, DLLNode*& tail) {
    if (node == head) head = node->next;
    if (node == tail) tail = node->prev;
    if (node->prev) node->prev->next = node->next;
    if (node->next) node->next->prev = node->prev;
    delete node;
}
```

### LRU Cache (Complete, Interview-Ready)

See the full implementation in the LRU Cache Design Pattern section above. The key components are:

```cpp
// Data structures:
//   unordered_map<int, CacheNode*> map   -- O(1) lookup by key
//   Doubly linked list with sentinels    -- O(1) insertion/removal
//
// Core helpers:
//   removeNode(node)    -- unlink node from current position
//   insertAtHead(node)  -- place node right after dummy head
//
// get(key):  map lookup -> move to head -> return value
// put(key):  if exists, update + move to head
//            if new + full, evict tail->prev, then insert at head
```

---

## 🔹 Related Concepts

- [[Singly Linked List]] -- simpler variant with one-directional traversal
- [[Arrays]] -- random access O(1) but O(n) insertion/deletion in the middle
- [[Deque]] -- efficiently implemented using a doubly linked list
- [[Hash Map and Hash Set]] -- combined with DLL for LRU Cache
- [[Stack]] -- can be implemented with either singly or doubly linked list
- [[Queue]] -- doubly linked list gives O(1) enqueue and dequeue
