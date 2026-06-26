---
tags:
  - algorithms
  - data-structure
  - deque
---

## 🔹 Real-World Analogy

Think of a **deck of cards held in your hand**. You can add or remove a card from the **top** or the **bottom** of the deck. Unlike a stack (top only) or a queue (back-in, front-out), a deque gives you full access to both ends.

Another way to picture it: a **double-ended tunnel**. People can enter or exit from either side.

```
         DEQUE (Double-Ended Queue)

       pop_front()                    pop_back()
      peek_front()                   peek_back()
            |                            |
            v                            v
    <-- [ front | ... | ... | ... | back ] -->
            ^                            ^
            |                            |
      push_front()                  push_back()


    Both ends are open. O(1) insert and remove at either end.
```

The key insight: a deque is the **most flexible** of the three linear structures. A stack restricts you to one end. A queue forces one-way flow. A deque removes all restrictions — you choose which end to use for each operation.

---

## 🔹 Definition

A **deque** (pronounced "deck") stands for **double-ended queue**. It is a linear data structure that supports insertion and removal of elements from **both the front and the back** in O(1) time.

- Generalization of both [[Stack]] and [[Queue]]
- Does NOT imply any ordering (it is not sorted)
- The name comes from "double-ended queue," not from "deck of cards" — though the card analogy works perfectly

---

## 🔹 Core Operations

All operations are O(1):

| Operation         | Description                              |
| ----------------- | ---------------------------------------- |
| `push_front(x)`   | Insert element `x` at the front          |
| `push_back(x)`    | Insert element `x` at the back           |
| `pop_front()`     | Remove and return the front element      |
| `pop_back()`      | Remove and return the back element       |
| `peek_front()`    | Return the front element without removal |
| `peek_back()`     | Return the back element without removal  |
| `isEmpty()`       | Return true if the deque has no elements |
| `size()`          | Return the number of elements            |

```
    push_front(7):

        BEFORE:  [ 3 | 5 | 9 ]
        AFTER:   [ 7 | 3 | 5 | 9 ]

    pop_back():

        BEFORE:  [ 7 | 3 | 5 | 9 ]
        AFTER:   [ 7 | 3 | 5 ]       returns 9
```

---

## 🔹 Implementation: Circular Array

The circular array approach extends the same idea used for a [[Queue]]. We maintain a `front` and `back` index into a fixed-size array, wrapping around when we hit the boundary. The difference from a plain circular queue is that **both indices can move in both directions**.

```
    Circular Array (capacity = 8):

    Index:   0   1   2   3   4   5   6   7
           +---+---+---+---+---+---+---+---+
           |   |   | 3 | 7 | 2 | 5 |   |   |
           +---+---+---+---+---+---+---+---+
                     ^               ^
                   front            back

    push_front(9):  front = (front - 1 + cap) % cap  -->  index 1
    push_back(8):   back  = (back + 1) % cap          -->  index 6

    After both:
    Index:   0   1   2   3   4   5   6   7
           +---+---+---+---+---+---+---+---+
           |   | 9 | 3 | 7 | 2 | 5 | 8 |   |
           +---+---+---+---+---+---+---+---+
                 ^                       ^
               front                    back
```

Key formula for wrapping:
- Move forward: `(index + 1) % capacity`
- Move backward: `(index - 1 + capacity) % capacity`

```cpp
class ArrayDeque {
    int* arr;
    int front, back, count, capacity;

public:
    ArrayDeque(int cap) : capacity(cap), front(0), back(0), count(0) {
        arr = new int[capacity];
    }

    void push_front(int x) {
        // Move front backward, then place element
        front = (front - 1 + capacity) % capacity;
        arr[front] = x;
        count++;
    }

    void push_back(int x) {
        // Place element at back, then move back forward
        arr[back] = x;
        back = (back + 1) % capacity;
        count++;
    }

    int pop_front() {
        int val = arr[front];
        front = (front + 1) % capacity;
        count--;
        return val;
    }

    int pop_back() {
        back = (back - 1 + capacity) % capacity;
        count--;
        return arr[back];
    }

    int peek_front() { return arr[front]; }
    int peek_back()  { return arr[(back - 1 + capacity) % capacity]; }
    bool isEmpty()   { return count == 0; }
    int size()       { return count; }
};
```

---

## 🔹 Implementation: Doubly Linked List

A [[Doubly Linked List]] is a natural fit for a deque. Each node has `prev` and `next` pointers, so inserting or removing at either end is O(1) with no wrapping logic.

```
    Doubly Linked List Deque:

    NULL <-- [ 3 ] <--> [ 7 ] <--> [ 2 ] <--> [ 5 ] --> NULL
              ^                                  ^
            head                                tail
           (front)                             (back)

    push_front(9):  new node before head, update head
    push_back(8):   new node after tail, update tail
    pop_front():    remove head, advance head to head->next
    pop_back():     remove tail, retreat tail to tail->prev
```

```cpp
struct Node {
    int data;
    Node* prev;
    Node* next;
    Node(int d) : data(d), prev(nullptr), next(nullptr) {}
};

class LinkedDeque {
    Node* head;
    Node* tail;
    int count;

public:
    LinkedDeque() : head(nullptr), tail(nullptr), count(0) {}

    void push_front(int x) {
        Node* node = new Node(x);
        node->next = head;
        if (head) head->prev = node;
        head = node;
        if (!tail) tail = node;
        count++;
    }

    void push_back(int x) {
        Node* node = new Node(x);
        node->prev = tail;
        if (tail) tail->next = node;
        tail = node;
        if (!head) head = node;
        count++;
    }

    int pop_front() {
        int val = head->data;
        Node* old = head;
        head = head->next;
        if (head) head->prev = nullptr;
        else tail = nullptr;
        delete old;
        count--;
        return val;
    }

    int pop_back() {
        int val = tail->data;
        Node* old = tail;
        tail = tail->prev;
        if (tail) tail->next = nullptr;
        else head = nullptr;
        delete old;
        count--;
        return val;
    }

    int peek_front() { return head->data; }
    int peek_back()  { return tail->data; }
    bool isEmpty()   { return count == 0; }
    int size()       { return count; }
};
```

---

## 🔹 Implementation Comparison

| Aspect              | Circular Array                  | Doubly Linked List             |
| ------------------- | ------------------------------- | ------------------------------ |
| Time (all ops)      | O(1) amortized                  | O(1) worst-case                |
| Memory overhead     | Low (contiguous block)          | High (two pointers per node)   |
| Cache performance   | Excellent (spatial locality)    | Poor (nodes scattered in heap) |
| Resizing            | Needed when full (O(n) copy)    | Never needed                   |
| Implementation      | Trickier (modular arithmetic)   | Simpler (pointer manipulation) |

In practice, the circular array version is usually preferred for performance due to cache locality, unless you need guaranteed O(1) worst-case (no amortized resizing).

---

## 🔹 Deque as a Generalization of Stack and Queue

A deque is the **universal linear adapter**. By restricting which operations you use, it becomes a stack or a queue:

```
    DEQUE
    ├── Use push_back() + pop_back()       -->  STACK (LIFO)
    ├── Use push_back() + pop_front()      -->  QUEUE (FIFO)
    └── Use all four push/pop operations   -->  Full DEQUE
```

| Restriction                            | Behavior       |
| -------------------------------------- | -------------- |
| Only `push_back` + `pop_back`          | [[Stack]]      |
| Only `push_back` + `pop_front`         | [[Queue]]      |
| Only `push_front` + `pop_front`        | [[Stack]]      |
| Only `push_front` + `pop_back`         | [[Queue]]      |
| All operations                         | Full Deque     |

This is why C++ `std::deque` is the default underlying container for both `std::stack` and `std::queue`.

---

## 🔹 Key Use Case: Sliding Window Maximum (Monotonic Deque)

This is THE classic deque problem. It appears constantly in interviews and competitive programming.

**Problem:** Given an array `nums` and a window size `k`, find the maximum element in every contiguous window of size `k`.

```
    Input:  nums = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3
    Output: [3, 3, 5, 5, 6, 7]

    Windows:
    [1  3  -1] -3  5  3  6  7   -->  max = 3
     1 [3  -1  -3] 5  3  6  7   -->  max = 3
     1  3 [-1  -3  5] 3  6  7   -->  max = 5
     1  3  -1 [-3  5  3] 6  7   -->  max = 5
     1  3  -1  -3 [5  3  6] 7   -->  max = 6
     1  3  -1  -3  5 [3  6  7]  -->  max = 7
```

**Why a deque?** We need three capabilities in one structure:
1. **Add to back** -- new elements enter the window from the right
2. **Remove from front** -- elements leave the window from the left (out of range)
3. **Remove from back** -- discard elements smaller than the new one (they can never be the max)

No other data structure provides all three in O(1).

**Core idea (Monotonic Deque):** Maintain a deque of **indices** where the corresponding values are in **decreasing order**. The front of the deque always holds the index of the current window maximum.

Two rules enforced at every step:
1. **Back cleanup:** Before adding a new element, pop all indices from the back whose values are less than or equal to the new element (they are useless -- the new element is bigger and will leave the window later).
2. **Front cleanup:** If the front index is outside the current window, pop it from the front.

---

## 🔹 Sliding Window Maximum: Step-by-Step Walkthrough

```
    Array: [1, 3, -1, -3, 5, 3, 6, 7]    k = 3
    Index:  0  1   2   3  4  5  6  7

    The deque stores INDICES. Values shown in parentheses for clarity.
    Deque front is LEFT, back is RIGHT.
```

**Step 0: i=0, nums[0]=1**
```
    Array:  [1] 3  -1  -3  5  3  6  7
             ^
    Deque is empty, push index 0.
    Deque:  [ 0(1) ]
    Window not yet full (need k=3 elements). No output.
```

**Step 1: i=1, nums[1]=3**
```
    Array:  [1   3] -1  -3  5  3  6  7
             ^   ^
    Back cleanup: nums[0]=1 <= nums[1]=3, so pop 0 from back.
    Push index 1.
    Deque:  [ 1(3) ]
    Window not yet full. No output.
```

**Step 2: i=2, nums[2]=-1**
```
    Array:  [1   3  -1] -3  5  3  6  7
             ^   ^   ^
             window
    Back cleanup: nums[1]=3 > nums[2]=-1, so keep 1. Push index 2.
    Deque:  [ 1(3),  2(-1) ]      <-- decreasing order maintained
    Front cleanup: index 1 >= i-k+1 = 0, so front is valid.
    Window full! Output nums[deque.front()] = nums[1] = 3

    OUTPUT: [3]
```

**Step 3: i=3, nums[3]=-3**
```
    Array:   1  [3  -1  -3] 5  3  6  7
                 ^   ^   ^
                  window
    Back cleanup: nums[2]=-1 > nums[3]=-3, so keep 2. Push index 3.
    Deque:  [ 1(3),  2(-1),  3(-3) ]
    Front cleanup: index 1 >= i-k+1 = 1, so front is valid.
    Output nums[1] = 3

    OUTPUT: [3, 3]
```

**Step 4: i=4, nums[4]=5**
```
    Array:   1   3  [-1  -3   5] 3  6  7
                      ^   ^   ^
                       window
    Back cleanup: nums[3]=-3 <= 5, pop 3. nums[2]=-1 <= 5, pop 2.
                  nums[1]=3 <= 5, pop 1. Deque empty. Push index 4.
    Deque:  [ 4(5) ]
    Front cleanup: index 4 >= i-k+1 = 2, valid.
    Output nums[4] = 5

    OUTPUT: [3, 3, 5]
```

**Step 5: i=5, nums[5]=3**
```
    Array:   1   3  -1  [-3   5   3] 6  7
                          ^   ^   ^
                           window
    Back cleanup: nums[4]=5 > nums[5]=3, keep 4. Push index 5.
    Deque:  [ 4(5),  5(3) ]
    Front cleanup: index 4 >= i-k+1 = 3, valid.
    Output nums[4] = 5

    OUTPUT: [3, 3, 5, 5]
```

**Step 6: i=6, nums[6]=6**
```
    Array:   1   3  -1  -3  [5   3   6] 7
                              ^   ^   ^
                               window
    Back cleanup: nums[5]=3 <= 6, pop 5. nums[4]=5 <= 6, pop 4.
                  Deque empty. Push index 6.
    Deque:  [ 6(6) ]
    Front cleanup: index 6 >= i-k+1 = 4, valid.
    Output nums[6] = 6

    OUTPUT: [3, 3, 5, 5, 6]
```

**Step 7: i=7, nums[7]=7**
```
    Array:   1   3  -1  -3   5  [3   6   7]
                                  ^   ^   ^
                                   window
    Back cleanup: nums[6]=6 <= 7, pop 6. Deque empty. Push index 7.
    Deque:  [ 7(7) ]
    Front cleanup: index 7 >= i-k+1 = 5, valid.
    Output nums[7] = 7

    OUTPUT: [3, 3, 5, 5, 6, 7]     <-- FINAL ANSWER
```

Summary of how the deque evolved:

```
    i=0:  Deque = [0(1)]
    i=1:  Deque = [1(3)]                 popped 0 from back
    i=2:  Deque = [1(3), 2(-1)]          output: 3
    i=3:  Deque = [1(3), 2(-1), 3(-3)]   output: 3
    i=4:  Deque = [4(5)]                 popped 3,2,1 from back. output: 5
    i=5:  Deque = [4(5), 5(3)]           output: 5
    i=6:  Deque = [6(6)]                 popped 5,4 from back. output: 6
    i=7:  Deque = [7(7)]                 popped 6 from back. output: 7
```

Notice: every element is pushed exactly once and popped at most once. Total work across all steps = O(n).

---

## 🔹 Template: Sliding Window Maximum

```cpp
#include <vector>
#include <deque>
using namespace std;

vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    vector<int> result;
    deque<int> dq;  // stores INDICES, not values

    for (int i = 0; i < nums.size(); i++) {

        // 1. Remove indices that have fallen out of the window
        //    The front of the deque might be outside [i-k+1, i]
        if (!dq.empty() && dq.front() < i - k + 1) {
            dq.pop_front();
        }

        // 2. Remove indices from the back whose values are <= nums[i]
        //    They can never be the max while nums[i] is in the window
        //    (nums[i] is bigger AND stays in the window longer)
        while (!dq.empty() && nums[dq.back()] <= nums[i]) {
            dq.pop_back();
        }

        // 3. Add current index to the back
        dq.push_back(i);

        // 4. Output the max for this window (front of deque)
        //    Only start outputting once we have a full window (i >= k-1)
        if (i >= k - 1) {
            result.push_back(nums[dq.front()]);
        }
    }

    return result;
}
```

**Why this is O(n):** Each index is pushed onto the deque exactly once and popped at most once. The while loop in step 2 may pop multiple elements in a single iteration, but across all iterations, the total number of pops cannot exceed n. So the total work is O(n), not O(nk).

---

## 🔹 Other Use Cases

**Palindrome Checking:**
Compare front and back characters, popping both ends inward. If all pairs match, the string is a palindrome.

```
    Input: "racecar"

    Deque:  [ r, a, c, e, c, a, r ]

    Step 1: peek_front()='r' == peek_back()='r' --> pop both
    Deque:  [ a, c, e, c, a ]

    Step 2: peek_front()='a' == peek_back()='a' --> pop both
    Deque:  [ c, e, c ]

    Step 3: peek_front()='c' == peek_back()='c' --> pop both
    Deque:  [ e ]

    One element left --> palindrome confirmed.
```

**Work-Stealing Algorithms (concurrency):**
In multi-threaded systems, each thread has its own deque of tasks:
- A thread pushes and pops work from its **own end** (like a stack -- LIFO for cache locality)
- When a thread runs out of work, it **steals** from the **other end** of another thread's deque

This minimizes contention because owner and thief operate on opposite ends.

**Undo/Redo with bounded history:**
Use a deque to store actions. Push new actions to the back. When the history exceeds a limit, pop the oldest from the front. Pop from the back to undo.

**Implementing both Stack and Queue with one structure:**
When you need both LIFO and FIFO behavior in the same program, a single deque can serve both roles without maintaining two separate data structures.

---

## 🔹 Time Complexity

| Operation       | Circular Array       | Doubly Linked List |
| --------------- | -------------------- | ------------------ |
| `push_front`    | O(1) amortized       | O(1)               |
| `push_back`     | O(1) amortized       | O(1)               |
| `pop_front`     | O(1)                 | O(1)               |
| `pop_back`      | O(1)                 | O(1)               |
| `peek_front`    | O(1)                 | O(1)               |
| `peek_back`     | O(1)                 | O(1)               |
| `isEmpty`       | O(1)                 | O(1)               |
| `size`          | O(1)                 | O(1)               |
| Random access   | O(1)                 | O(n)               |
| Space           | O(n)                 | O(n) + pointer overhead |

The "amortized" note for circular array push operations accounts for the rare O(n) resize when the array is full. If the capacity is pre-allocated or the problem has a known bound, all operations are strict O(1).

---

## 🔹 Common Pitfalls

1. **Name confusion:** "Deque" is pronounced "deck," not "dee-queue." It is not a "double-ended stack" -- it is a double-ended queue. The abbreviation is "deque," not "dequeue" (which sounds like a verb meaning "remove from queue").

2. **Monotonic deque mistakes:** When implementing sliding window maximum, the most common bugs are:
   - Storing **values** instead of **indices** in the deque (you need indices to check if the front has left the window)
   - Using `<` instead of `<=` in the back cleanup (if you use `<`, duplicate maxima can cause issues depending on the problem)
   - Forgetting to check `i >= k - 1` before recording output (outputting before the window is full)
   - Checking the front **after** pushing the new index instead of before (the new index is always valid, but checking order matters for clarity)

3. **Circular array off-by-one errors:** The modular arithmetic for `push_front` (decrement then write) vs `push_back` (write then increment) must be consistent. Mixing conventions causes overwrites or gaps.

4. **Forgetting resize logic:** A circular array deque without resize logic will silently overwrite data or crash when full. Always either resize or enforce a max capacity.

---

## 🔹 Related Concepts

- [[Stack]] -- LIFO subset of deque (one-end operations only)
- [[Queue]] -- FIFO subset of deque (opposite-end operations)
- [[Arrays]] -- underlying storage for circular array implementation
- [[Doubly Linked List]] -- underlying storage for linked list implementation
- [[Min Heap and Max Heap]] -- alternative for finding max/min, but O(log n) per operation vs O(1) amortized for monotonic deque in sliding window problems
- [[Binary Search Tree]] -- another alternative for ordered access, but overkill for sliding window problems
