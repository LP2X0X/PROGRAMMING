---
tags:
  - design
  - pattern
  - structural
  - flyweight
---

## Intent

Uses **sharing** to support large numbers of fine-grained objects efficiently.

## Key Concept

Separates intrinsic state (shared, immutable) from extrinsic state (unique, passed in by client).

## When to Use

- When your application creates a huge number of similar objects that eat up memory (e.g., characters in a text editor, trees in a game forest).

## Example (C++)

```cpp
// Each unique tree type is shared (flyweight)
class TreeType {
    std::string name, color, texture;
public:
    TreeType(std::string n, std::string c, std::string t)
        : name(n), color(c), texture(t) {}
    void draw(int x, int y) { /* draw at position using shared data */ }
};

// Extrinsic state: position is unique per tree
class Tree {
    int x, y;
    TreeType* type; // shared
public:
    Tree(int x, int y, TreeType* t) : x(x), y(y), type(t) {}
    void draw() { type->draw(x, y); }
};
```
