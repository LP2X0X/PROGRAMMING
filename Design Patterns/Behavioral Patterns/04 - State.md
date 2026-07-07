---
tags: [design, pattern, behavioral, state]
---

## Overview

Allows an object to **alter its behavior when its internal state changes**. The object will appear to change its class.

Instead of massive `if/else` or `switch` blocks checking state, each state becomes its own class.

## When to Use

- When an object's behavior depends on its state and it must change behavior at runtime
- When operations have large conditional statements that depend on the object's state

## Related Patterns

- [[02 - Strategy]] (Strategy swaps algorithms by client choice, State swaps behavior automatically based on internal state)

## C++ Example

```cpp
class State {
public:
    virtual void handle() = 0;
};

class LockedState : public State {
public:
    void handle() override { /* show "enter PIN" screen */ }
};

class UnlockedState : public State {
public:
    void handle() override { /* show home screen */ }
};

class Phone {
    State* state;
public:
    void setState(State* s) { state = s; }
    void pressHome() { state->handle(); }
};
```
