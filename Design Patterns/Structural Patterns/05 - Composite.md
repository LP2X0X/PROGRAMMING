---
tags:
  - design
  - pattern
  - structural
  - composite
---

## Intent

Composes objects into **tree structures** to represent part-whole hierarchies. Lets clients treat individual objects and compositions of objects uniformly.

## When to Use

- When you have a tree-like structure (file system, UI components, organization charts) and want to treat leaves and containers the same way.

## Example (C++)

```cpp
class FileComponent {
public:
    virtual void showDetails() = 0;
};

class File : public FileComponent {
    std::string name;
public:
    File(std::string n) : name(n) {}
    void showDetails() override { /* print file name */ }
};

class Folder : public FileComponent {
    std::string name;
    std::vector<FileComponent*> children;
public:
    Folder(std::string n) : name(n) {}
    void add(FileComponent* c) { children.push_back(c); }
    void showDetails() override {
        for (auto* child : children)
            child->showDetails(); // works on files AND folders
    }
};
```
