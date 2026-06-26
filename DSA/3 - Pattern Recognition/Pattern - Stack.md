---
tags:
  - algorithms
  - pattern-recognition
  - stack
---

## 🔹 When to Suspect This Pattern

- **Matching/nesting** — brackets, parentheses, HTML tags, nested structures
- **"Next greater/smaller element"** — monotonic relationships in an array
- **LIFO processing** — undo operations, backtracking simulation, history tracking
- **Expression evaluation** — infix, postfix, calculator problems
- Keywords: "valid parentheses", "decode string", "daily temperatures", "largest rectangle"
- A recursive solution exists, and you need to convert it to iterative

---

## 🔹 Confirming It's the Right Pattern

- [ ] Does the problem involve **matching pairs** (open/close)?
- [ ] Do you need to find the **nearest** greater/smaller element for each position?
- [ ] Is there a **last-in-first-out** processing order?
- [ ] Does the problem involve **nesting depth** or **unwinding** nested structures?
- [ ] Can you solve it by "processing elements and deferring decisions to later"?

---

## 🔹 Key Variants

### Variant 1: Matching Brackets / Parentheses

```cpp
bool isValid(string s) {
    stack<char> stk;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{')
            stk.push(c);
        else {
            if (stk.empty()) return false;
            char top = stk.top(); stk.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{'))
                return false;
        }
    }
    return stk.empty();
}
```

### Variant 2: Monotonic Stack (Next Greater Element)

Maintain a stack where elements are in **monotonically increasing** (or decreasing) order.

```cpp
// Next Greater Element for each index
vector<int> nextGreater(n, -1);
stack<int> stk;  // stores indices

for (int i = 0; i < n; i++) {
    while (!stk.empty() && arr[stk.top()] < arr[i]) {
        nextGreater[stk.top()] = arr[i];
        stk.pop();
    }
    stk.push(i);
}
```

> [!tip] Monotonic Stack Direction
> - **Next Greater**: use a stack that maintains **decreasing** order (pop when current > top)
> - **Next Smaller**: use a stack that maintains **increasing** order (pop when current < top)
> - **Previous Greater/Smaller**: iterate right-to-left, or adjust the logic

### Variant 3: Expression Evaluation

```cpp
// Basic calculator: handle +, -, (, )
stack<int> nums;
stack<int> signs;
int result = 0, sign = 1, num = 0;
for (char c : s) {
    if (isdigit(c)) {
        num = num * 10 + (c - '0');
    } else {
        result += sign * num;
        num = 0;
        if (c == '+') sign = 1;
        else if (c == '-') sign = -1;
        else if (c == '(') {
            nums.push(result);
            signs.push(sign);
            result = 0; sign = 1;
        } else if (c == ')') {
            result = nums.top() + signs.top() * result;
            nums.pop(); signs.pop();
        }
    }
}
result += sign * num;
```

---

## 🔹 Classic Problems

| Problem | Variant | Key Insight |
|---|---|---|
| **Valid Parentheses** | Matching | Push open, pop and match on close |
| **Daily Temperatures** | Monotonic (decreasing) | Stack stores indices; pop when warmer day found |
| **Next Greater Element** | Monotonic (decreasing) | Classic monotonic stack template |
| **Largest Rectangle in Histogram** | Monotonic (increasing) | Pop when bar is shorter; calculate area |
| **Trapping Rain Water** | Monotonic (decreasing) or two-pass | Stack tracks boundaries for water |
| **Decode String** | Nesting | Stack stores (count, string) pairs |
| **Basic Calculator** | Expression | Stacks for numbers and operators |
| **Min Stack** | Augmented | Secondary stack tracks running minimum |

---

## 🔹 Common Mistakes

- **Forgetting to check `stk.empty()` before `stk.top()`**: accessing top of empty stack is undefined behavior
- **Storing values vs indices**: for "next greater element" problems, store **indices** (you can always get the value from the array)
- **Monotonic direction confusion**: "next greater" = pop when current is greater = stack is *decreasing*. Think: "what stays on the stack?"
- **Not handling remaining elements**: after the loop, elements still on the stack have no next greater/smaller — handle them (usually set to -1 or n)
- **Histogram / trapping water**: these are hard. The key is understanding what each pop represents (a completed rectangle or water unit)

---

## 🔹 When Stack vs Other

| Scenario | Stack? | Alternative |
|---|---|---|
| Matching brackets | **Yes** | — |
| Next greater element | **Yes** (monotonic) | Brute force O(n^2) |
| Min/max in sliding window | **Monotonic Deque** | Deque is better (double-ended) |
| BFS/DFS simulation | Stack for DFS | Queue for BFS |
| Undo/redo | **Yes** | Two stacks (undo + redo) |

---

## 🔹 Related Patterns

- [[Pattern - Array and String]] — monotonic stack often processes arrays
- [[Pattern - Two Pointers]] — trapping rain water can use either approach
- [[Pattern - Dynamic Programming]] — largest rectangle can also be solved with DP
- [[How to Pick the Right Data Structure]] — stack vs queue vs deque decision
