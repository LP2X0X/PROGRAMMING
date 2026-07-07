---
tags:
  - design
  - pattern
  - structural
  - proxy
---

## Intent

Provides a **surrogate or placeholder** for another object to control access to it.

## Types of Proxies

- **Virtual** -- lazy initialization
- **Protection** -- access control
- **Remote** -- represents an object in a different address space
- **Logging/Caching** proxy

## When to Use

- Lazy loading of expensive objects.
- Access control, logging, caching.
- Working with remote services.

## Related Patterns

- [[02 - Decorator]] (both wrap objects, but Proxy controls access while Decorator adds behavior)

## Example (C++)

```cpp
class Image {
public:
    virtual void display() = 0;
};

class RealImage : public Image {
    std::string filename;
public:
    RealImage(std::string f) : filename(f) {
        // expensive: loads image from disk
    }
    void display() override { /* render image */ }
};

class ProxyImage : public Image {
    RealImage* realImage = nullptr;
    std::string filename;
public:
    ProxyImage(std::string f) : filename(f) {}
    void display() override {
        if (!realImage)
            realImage = new RealImage(filename); // lazy load
        realImage->display();
    }
};
```
