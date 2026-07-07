---
tags: [design, pattern, behavioral, chain-of-responsibility]
---

## Overview

Passes a request along a **chain of handlers**. Each handler decides either to process the request or to pass it to the next handler in the chain.

## When to Use

- When multiple objects may handle a request and the handler isn't known in advance
- When you want to decouple sender from receiver (e.g., middleware pipelines, event bubbling, logging levels)

## Related Patterns

- [[03 - Command]] (Command encapsulates a request as an object, Chain of Responsibility routes it through handlers)

## C++ Example

```cpp
class Handler {
protected:
    Handler* next = nullptr;
public:
    void setNext(Handler* h) { next = h; }
    virtual void handle(int request) {
        if (next) next->handle(request);
    }
};

class LowLevelSupport : public Handler {
public:
    void handle(int severity) override {
        if (severity < 5) { /* handle it */ }
        else Handler::handle(severity); // pass up the chain
    }
};

class HighLevelSupport : public Handler {
public:
    void handle(int severity) override {
        if (severity >= 5) { /* handle it */ }
        else Handler::handle(severity);
    }
};
```
