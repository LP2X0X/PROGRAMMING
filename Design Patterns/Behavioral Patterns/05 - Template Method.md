---
tags: [design, pattern, behavioral, template-method]
---

## Overview

Defines the **skeleton of an algorithm** in a base class, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

## When to Use

- When multiple classes share the same algorithm structure but differ in specific steps

## Related Patterns

- [[02 - Strategy]] (Strategy replaces the entire algorithm, Template Method customizes steps within a fixed skeleton)

## C++ Example

```cpp
class DataMiner {
public:
    // Template method - defines the skeleton
    void mine() {
        openFile();
        extractData();
        parseData();
        analyzeData();
        closeFile();
    }
protected:
    virtual void openFile() = 0;
    virtual void extractData() = 0;
    void parseData() { /* common parsing logic */ }
    void analyzeData() { /* common analysis logic */ }
    virtual void closeFile() = 0;
};

class CSVDataMiner : public DataMiner {
protected:
    void openFile() override { /* open CSV */ }
    void extractData() override { /* read CSV rows */ }
    void closeFile() override { /* close CSV */ }
};
```
