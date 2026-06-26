---
tags:
  - algorithms
  - data-structure
  - stack
---
## 🔹 What Is a Stack?

A **stack** is a linear data structure that follows the **LIFO** (Last In, First Out) principle: the last element added is the first one removed.

**Real-world analogy:** Think of a stack of plates in a cafeteria. You can only take the top plate off, and you can only add a new plate on top. You never pull from the middle or bottom.

```
        ┌─────────┐
        │  push ──►│ ◄── top (most recently added)
        ├─────────┤
        │  item C  │
        ├─────────┤
        │  item B  │
        ├─────────┤
        │  item A  │ ◄── bottom (first added, last removed)
        └─────────┘

    push(D):                pop():
        ┌─────────┐            ┌─────────┐
   ──►  │  item D  │ ◄─ top    │ item D  │ ──► removed & returned
        ├─────────┤            ├─────────┤
        │  item C  │            │  item C  │ ◄─ new top
        ├─────────┤            ├─────────┤
        │  item B  │            │  item B  │
        ├─────────┤            ├─────────┤
        │  item A  │            │  item A  │
        └─────────┘            └─────────┘
```

Key contrast with [[Queue]]: a queue is FIFO (First In, First Out). A stack is LIFO. Mixing these up is a classic mistake.

---

## 🔹 Core Operations

All fundamental stack operations are **O(1)** -- constant time:

| Operation      | Description                        | Time |
| -------------- | ---------------------------------- | ---- |
| `push(item)`   | Add item to the top                | O(1) |
| `pop()`        | Remove and return the top item     | O(1) |
| `peek()` / `top()` | View top item without removing | O(1) |
| `isEmpty()`    | Check if the stack has no elements | O(1) |
| `size()`       | Return the number of elements      | O(1) |

There is no O(n) traversal, no searching, no random access. If you need those, a stack is the wrong data structure.

---

## 🔹 Implementation: Array-Based

Use a dynamic array ([[Arrays]]) with a `top` index that tracks the position of the topmost element. This is the most common implementation in practice.

```
    Array:  [ A | B | C |   |   |   ]
    Index:    0   1   2   3   4   5
                      ^
                     top = 2

    After push(D):
            [ A | B | C | D |   |   ]
              0   1   2   3   4   5
                          ^
                         top = 3

    After pop() -> returns D:
            [ A | B | C |   |   |   ]
              0   1   2   3   4   5
                      ^
                     top = 2
```

```cpp
class ArrayStack {
    int* arr;
    int capacity;
    int topIdx;

public:
    ArrayStack(int cap) : capacity(cap), topIdx(-1) {
        arr = new int[capacity];
    }

    void push(int item) {
        if (topIdx == capacity - 1) {
            // resize or throw overflow
            return;
        }
        arr[++topIdx] = item;
    }

    int pop() {
        if (isEmpty()) throw runtime_error("Stack underflow");
        return arr[topIdx--];
    }

    int peek() {
        if (isEmpty()) throw runtime_error("Stack is empty");
        return arr[topIdx];
    }

    bool isEmpty() { return topIdx == -1; }
    int size()     { return topIdx + 1; }
};
```

**Pros:** Cache-friendly (contiguous memory), less memory overhead per element.
**Cons:** Fixed capacity unless you implement dynamic resizing (amortized O(1) for push, but occasional O(n) copy).

---

## 🔹 Implementation: Linked-List-Based

Use a [[Singly Linked List]] where **head = top of stack**. Push inserts at head, pop removes from head -- both O(1) with no resizing.

```
    push(C):
                  ┌───┐    ┌───┐    ┌───┐
      top ──►     │ C │───►│ B │───►│ A │───► null
                  └───┘    └───┘    └───┘

    pop() -> returns C:
                  ┌───┐    ┌───┐
      top ──►     │ B │───►│ A │───► null
                  └───┘    └───┘
```

```cpp
struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};

class LinkedStack {
    Node* top;
    int count;

public:
    LinkedStack() : top(nullptr), count(0) {}

    void push(int item) {
        Node* node = new Node(item);
        node->next = top;
        top = node;
        count++;
    }

    int pop() {
        if (isEmpty()) throw runtime_error("Stack underflow");
        int val = top->data;
        Node* temp = top;
        top = top->next;
        delete temp;
        count--;
        return val;
    }

    int peek() {
        if (isEmpty()) throw runtime_error("Stack is empty");
        return top->data;
    }

    bool isEmpty() { return top == nullptr; }
    int size()     { return count; }
};
```

**Pros:** No capacity limit, no resizing, always O(1) push/pop.
**Cons:** Extra memory per node (pointer overhead), not cache-friendly (scattered heap allocations).

### Comparison

| Aspect             | Array-Based           | Linked-List-Based       |
| ------------------ | --------------------- | ----------------------- |
| Memory layout      | Contiguous            | Scattered (heap nodes)  |
| Cache performance  | Excellent             | Poor                    |
| Memory overhead    | Low (just the array)  | High (pointer per node) |
| Resizing           | Needed (amortized)    | Not needed              |
| Max size           | Pre-defined or dynamic| Unbounded (until OOM)   |
| Typical use        | Most real code / STL  | When size is unknown    |

In practice, almost every standard library stack uses a dynamic array underneath (e.g., C++ `std::stack` defaults to `std::deque`, Python `list`, C# `Stack<T>`).

---

## 🔹 The Call Stack

Every running program has a **call stack** managed by the OS/runtime. When a function is called, a **stack frame** is pushed; when it returns, the frame is popped.

Each stack frame contains:
- Local variables
- Function parameters
- Return address (where to resume after the function ends)
- Saved registers

```
    main() calls funcA(), funcA() calls funcB():

    ┌──────────────────┐
    │  funcB()         │ ◄─ top of call stack (currently executing)
    │  locals: z = 30  │
    │  return addr ─────────► back to funcA, line 7
    ├──────────────────┤
    │  funcA()         │
    │  locals: y = 20  │
    │  return addr ─────────► back to main, line 4
    ├──────────────────┤
    │  main()          │
    │  locals: x = 10  │
    │  return addr ─────────► OS / program exit
    └──────────────────┘

    When funcB() returns, its frame is popped.
    Execution resumes in funcA() at the saved return address.
```

**Recursion** works by pushing a new frame for each recursive call. If recursion goes too deep (no base case, or input is too large), the call stack exhausts its memory and you get a **stack overflow**.

```
    factorial(5) -> factorial(4) -> factorial(3) -> ... -> factorial(0)

    ┌─────────────────┐
    │ factorial(0) = 1│ ◄─ base case reached, start unwinding
    ├─────────────────┤
    │ factorial(1)    │
    ├─────────────────┤
    │ factorial(2)    │
    ├─────────────────┤
    │ factorial(3)    │
    ├─────────────────┤
    │ factorial(4)    │
    ├─────────────────┤
    │ factorial(5)    │
    └─────────────────┘
```

This is why **iterative DFS with an explicit stack** is sometimes preferred over recursive DFS -- you control the stack size and avoid stack overflow on large inputs.

---

## 🔹 Common Uses in Algorithms

This is the highest-value section. Stacks appear everywhere in algorithm problems.

### 1. Parentheses / Bracket Matching

**Problem:** Given a string of brackets `()[]{}`, determine if they are balanced.

**Why a stack:** Every opening bracket must eventually be matched by the correct closing bracket, and matches must be nested properly (LIFO order). A stack naturally enforces this nesting.

See template code below for the full implementation.

### 2. Undo / Redo

- **Undo stack:** Each action is pushed. Undo pops the last action.
- **Redo stack:** When you undo, the undone action is pushed to a redo stack. Redo pops from there. Any new action clears the redo stack.

Browser back/forward buttons work the same way -- two stacks.

### 3. DFS (Depth-First Search)

Recursive DFS implicitly uses the call stack. **Iterative DFS** uses an explicit stack:

```cpp
void iterativeDFS(Graph& g, int start) {
    vector<bool> visited(g.size(), false);
    stack<int> s;
    s.push(start);

    while (!s.empty()) {
        int node = s.top(); s.pop();
        if (visited[node]) continue;
        visited[node] = true;
        process(node);

        for (int neighbor : g.adj[node]) {
            if (!visited[neighbor])
                s.push(neighbor);
        }
    }
}
```

See also: [[Trees]], [[Graphs]]

### 4. Expression Evaluation (Postfix / RPN)

Postfix expression `3 4 + 2 *` means `(3 + 4) * 2 = 14`.

Algorithm: scan left to right. Push numbers. When you hit an operator, pop two operands, apply the operator, push the result.

```
    Input: 3 4 + 2 *

    Step 1: push 3        Stack: [3]
    Step 2: push 4        Stack: [3, 4]
    Step 3: +  -> pop 4,3 -> push 7   Stack: [7]
    Step 4: push 2        Stack: [7, 2]
    Step 5: *  -> pop 2,7 -> push 14  Stack: [14]

    Result: 14
```

### 5. Next Greater Element / Next Smaller Element (Monotonic Stack)

This pattern is so important it gets its own section below.

### 6. Infix to Postfix Conversion (Shunting Yard)

Uses a stack to hold operators, respecting precedence and associativity. Operators with lower precedence get pushed; operators with higher or equal precedence cause pops.

---

## 🔹 Monotonic Stack Pattern

A **monotonic stack** maintains elements in either strictly increasing or strictly decreasing order from bottom to top. It is the go-to technique for "next greater element" and "next smaller element" problems.

**When to reach for a monotonic stack:** Whenever a problem asks "for each element, find the next/previous greater/smaller element," you almost certainly want a monotonic stack.

### How It Works (Decreasing Monotonic Stack for Next Greater Element)

Maintain a stack where elements decrease from bottom to top. When a new element is larger than the top, it is the "next greater element" for everything it pops off.

```
    Input:  [2, 1, 2, 4, 3]
    Goal:   For each element, find the next element to its right that is greater.
    Answer: [4, 2, 4, -1, -1]

    Walk through (stack stores indices, shown with values for clarity):

    i=0, val=2:  Stack empty, push.           Stack: [2]
    i=1, val=1:  1 < 2 (top), push.           Stack: [2, 1]
    i=2, val=2:  2 > 1 (top), pop 1 -> NGE[1]=2
                 2 = 2 (top), stop popping.
                 Push 2.                       Stack: [2, 2]
    i=3, val=4:  4 > 2 (top), pop 2 -> NGE[2]=4
                 4 > 2 (top), pop 2 -> NGE[0]=4
                 Stack empty, push.            Stack: [4]
    i=4, val=3:  3 < 4 (top), push.           Stack: [4, 3]

    End: remaining in stack have no next greater -> -1
         NGE[3] = -1, NGE[4] = -1

    Final: [4, 2, 4, -1, -1]
```

```
    Visualizing the monotonic (decreasing) stack during processing:

    ┌───┐
    │ 2 │           After i=0
    └───┘

    ┌───┐
    │ 1 │           After i=1 (1 < 2, so push)
    ├───┤
    │ 2 │
    └───┘

    ┌───┐
    │ 2 │           After i=2 (popped 1 because 2 > 1)
    ├───┤
    │ 2 │
    └───┘

    ┌───┐
    │ 4 │           After i=3 (popped both 2s because 4 > 2)
    └───┘

    ┌───┐
    │ 3 │           After i=4 (3 < 4, so push)
    ├───┤
    │ 4 │
    └───┘
```

**Key insight:** Each element is pushed once and popped at most once, so the total time complexity is **O(n)** even though there is a while loop inside the for loop.

**Variations:**
- **Increasing monotonic stack** -- for "next smaller element" problems
- **Previous greater/smaller** -- iterate from left to right and look at what is currently on the stack when you push
- **Circular arrays** -- iterate through the array twice (index `i % n`)

See template code below for the full implementation.

---

## 🔹 Time Complexity Summary

| Operation   | Array-Based | Linked-List-Based |
| ----------- | ----------- | ----------------- |
| push        | O(1)*       | O(1)              |
| pop         | O(1)        | O(1)              |
| peek / top  | O(1)        | O(1)              |
| isEmpty     | O(1)        | O(1)              |
| size        | O(1)        | O(1)              |
| Search      | O(n)        | O(n)              |

*Amortized O(1) for dynamic arrays (occasional O(n) resize).

**Space complexity:** O(n) for n elements in the stack.

---

## 🔹 Common Pitfalls

1. **Stack overflow from deep recursion** -- Every recursive call adds a frame to the call stack. If recursion depth is proportional to input size (e.g., traversing a linked list recursively), large inputs will crash. Convert to iterative with an explicit stack.

2. **Popping from an empty stack** -- Always check `isEmpty()` before `pop()` or `peek()`. In interviews, mention this edge case even if the problem guarantees valid input.

3. **Confusing LIFO with FIFO** -- Stack (LIFO): last in, first out. [[Queue]] (FIFO): first in, first out. If you process elements in the order they arrived, you need a queue, not a stack.

4. **Using the wrong monotonic stack direction** -- Decreasing stack for "next greater," increasing stack for "next smaller." Getting this backwards will produce incorrect results silently.

5. **Forgetting to handle remaining elements** -- After processing all input with a monotonic stack, elements still in the stack have no next greater/smaller. You must handle them (usually set result to -1).

---

## 🔹 Template: Parentheses Matching

This is one of the most common interview problems. Learn this pattern cold.

**Problem:** Given a string containing `(`, `)`, `{`, `}`, `[`, `]`, determine if the input is valid. An input string is valid if:
- Open brackets are closed by the same type.
- Open brackets are closed in the correct order.
- Every close bracket has a corresponding open bracket.

```cpp
#include <stack>
#include <string>
#include <unordered_map>
using namespace std;

bool isValid(string s) {
    stack<char> stk;

    // Map each closing bracket to its matching opening bracket.
    // This avoids a chain of if/else and scales to any bracket types.
    unordered_map<char, char> match = {
        {')', '('},
        {']', '['},
        {'}', '{'}
    };

    for (char c : s) {
        if (match.find(c) == match.end()) {
            // c is an opening bracket -- push it
            stk.push(c);
        } else {
            // c is a closing bracket
            // Check 1: is there an opening bracket to match?
            if (stk.empty()) return false;

            // Check 2: does the top of stack match this closing bracket?
            if (stk.top() != match[c]) return false;

            // Match found -- pop the opening bracket
            stk.pop();
        }
    }

    // If the stack is empty, every opener was matched.
    // If not empty, there are unmatched opening brackets.
    return stk.empty();
}
```

**Walkthrough with `"{[()]}"`:**

```
    Char   Action              Stack (top on right)
    ─────────────────────────────────────────────
    {      opening -> push     [ { ]
    [      opening -> push     [ {, [ ]
    (      opening -> push     [ {, [, ( ]
    )      closing, match (    [ {, [ ]
    ]      closing, match [    [ { ]
    }      closing, match {    [ ]

    Stack empty -> VALID
```

**Walkthrough with `"([)]"`:**

```
    Char   Action              Stack (top on right)
    ─────────────────────────────────────────────
    (      opening -> push     [ ( ]
    [      opening -> push     [ (, [ ]
    )      closing, top is [   MISMATCH -> return false
```

**Why this works:** The stack enforces the nesting constraint. The most recently opened bracket must be closed first -- which is exactly LIFO.

**Edge cases to mention in interviews:**
- Empty string -> valid (stack stays empty)
- Single bracket -> invalid (stack not empty at end, or empty when closing)
- Only opening brackets -> invalid (stack not empty at end)
- Only closing brackets -> invalid (stack empty when we try to match)

---

## 🔹 Template: Next Greater Element (Monotonic Stack)

**Problem:** Given an array of integers, find the next greater element for each element. The next greater element of an element `x` is the first element to the right of `x` that is greater than `x`. If none exists, output -1.

```cpp
#include <vector>
#include <stack>
using namespace std;

// Returns an array where result[i] = next greater element for nums[i],
// or -1 if no greater element exists to the right.
vector<int> nextGreaterElement(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);  // default: no next greater
    stack<int> stk;             // stores INDICES (not values)

    // Iterate left to right.
    // Maintain a decreasing monotonic stack (bottom to top).
    for (int i = 0; i < n; i++) {
        // While current element is greater than the element at
        // the index on top of the stack, we have found the
        // "next greater element" for that index.
        while (!stk.empty() && nums[i] > nums[stk.top()]) {
            int idx = stk.top();
            stk.pop();
            result[idx] = nums[i];  // nums[i] is the NGE for nums[idx]
        }

        // Push current index onto the stack
        stk.push(i);
    }

    // Elements remaining in the stack have no next greater element.
    // result[] was initialized to -1, so nothing more to do.

    return result;
}
```

**Example:**

```
    Input:  [2, 1, 2, 4, 3]
    Output: [4, 2, 4, -1, -1]

    i=0 (val=2): stack empty, push 0.                   Stack: [0]
    i=1 (val=1): 1 < nums[0]=2, push 1.                Stack: [0, 1]
    i=2 (val=2): 2 > nums[1]=1, pop 1, result[1]=2.
                 2 = nums[0]=2, not >, stop.
                 Push 2.                                 Stack: [0, 2]
    i=3 (val=4): 4 > nums[2]=2, pop 2, result[2]=4.
                 4 > nums[0]=2, pop 0, result[0]=4.
                 Stack empty, push 3.                    Stack: [3]
    i=4 (val=3): 3 < nums[3]=4, push 4.                Stack: [3, 4]

    Done. Indices 3,4 remain -> result[3]=-1, result[4]=-1.
    Result: [4, 2, 4, -1, -1]
```

**Variation -- Next Smaller Element:** Change the comparison from `>` to `<` and maintain an increasing monotonic stack:

```cpp
vector<int> nextSmallerElement(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);
    stack<int> stk;

    for (int i = 0; i < n; i++) {
        // Pop while current is SMALLER than top (increasing stack)
        while (!stk.empty() && nums[i] < nums[stk.top()]) {
            result[stk.top()] = nums[i];
            stk.pop();
        }
        stk.push(i);
    }
    return result;
}
```

**Time complexity:** O(n). Each element is pushed once and popped at most once.
**Space complexity:** O(n) for the stack and result array.

---

## 🔹 Related Concepts

- [[Queue]] -- FIFO counterpart; use when order of arrival matters
- [[Deque]] -- double-ended queue, can act as both stack and queue
- [[Arrays]] -- underlying structure for array-based stack
- [[Singly Linked List]] -- underlying structure for linked-list-based stack
- [[Trees]] -- DFS traversal uses a stack (explicitly or via recursion)
- [[Recursion]] -- recursive calls use the system call stack
- [[Depth-First Search]] -- iterative DFS replaces recursion with an explicit stack
- [[Monotonic Stack]] -- dedicated pattern for next greater/smaller problems
