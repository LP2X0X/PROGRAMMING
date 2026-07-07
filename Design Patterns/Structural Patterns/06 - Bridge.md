---
tags:
  - design
  - pattern
  - structural
  - bridge
---

## Intent

Decouples an **abstraction from its implementation** so the two can vary independently.

## Analogy

Think of it as: shape (abstraction) and color (implementation) can change independently without a `RedCircle`, `BlueCircle`, `RedSquare`, `BlueSquare` explosion.

## When to Use

- When both the abstraction and the implementation may change, and you want to avoid a combinatorial explosion of subclasses.

## Example (C++)

```cpp
// Implementation hierarchy
class Color {
public:
    virtual std::string fill() = 0;
};
class Red : public Color {
public:
    std::string fill() override { return "red"; }
};
class Blue : public Color {
public:
    std::string fill() override { return "blue"; }
};

// Abstraction hierarchy
class Shape {
protected:
    Color* color;
public:
    Shape(Color* c) : color(c) {}
    virtual void draw() = 0;
};
class Circle : public Shape {
public:
    Circle(Color* c) : Shape(c) {}
    void draw() override { /* draw circle filled with color->fill() */ }
};
```
