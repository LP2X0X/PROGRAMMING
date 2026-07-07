---
tags: [design, pattern, behavioral, strategy]
---

## Overview

Defines a **family of algorithms**, encapsulates each one, and makes them **interchangeable**. Strategy lets the algorithm vary independently from the clients that use it.

## When to Use

- When you have multiple ways of doing the same thing (sorting, compression, routing, payment) and want to switch between them at runtime

## Related Patterns

- [[04 - State]] (State changes behavior based on internal state, Strategy lets clients choose the behavior)

## C++ Example

```cpp
class SortStrategy {
public:
    virtual void sort(std::vector<int>& data) = 0;
};

class QuickSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override { /* quicksort */ }
};

class MergeSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override { /* mergesort */ }
};

class Sorter {
    SortStrategy* strategy;
public:
    void setStrategy(SortStrategy* s) { strategy = s; }
    void doSort(std::vector<int>& data) { strategy->sort(data); }
};
```
