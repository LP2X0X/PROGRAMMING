---
tags:
  - design
  - pattern
  - creational
  - prototype
---

## Overview

The Prototype pattern creates new objects by **cloning an existing object** (the prototype) instead of building from scratch.

## When to Use

- When object creation is expensive (e.g., requires database calls or complex computation), and a similar object already exists
- When you need copies that are independent of their concrete classes

## Example (C++)

```cpp
class Shape {
public:
    virtual Shape* clone() = 0;
    virtual void draw() = 0;
};

class Circle : public Shape {
    int radius;
public:
    Circle(int r) : radius(r) {}
    Shape* clone() override { return new Circle(*this); }
    void draw() override { /* draw circle */ }
};
```
