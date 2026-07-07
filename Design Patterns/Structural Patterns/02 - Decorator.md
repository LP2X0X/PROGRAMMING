---
tags:
  - design
  - pattern
  - structural
  - decorator
---

## Intent

Attaches **additional responsibilities** to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.

## How It Works

Decorators wrap the original object and add behavior before or after delegating to it.

## When to Use

- When you need to add behavior to individual objects without affecting other objects of the same class.
- When extension by subclassing is impractical (combinatorial explosion of subclasses).

## Related Patterns

- [[04 - Proxy]] (both wrap objects, but Proxy controls access while Decorator adds behavior)

## Example (C++)

```cpp
class Coffee {
public:
    virtual double cost() = 0;
    virtual std::string description() = 0;
};

class SimpleCoffee : public Coffee {
public:
    double cost() override { return 1.0; }
    std::string description() override { return "Simple coffee"; }
};

class MilkDecorator : public Coffee {
    Coffee* wrapped;
public:
    MilkDecorator(Coffee* c) : wrapped(c) {}
    double cost() override { return wrapped->cost() + 0.5; }
    std::string description() override {
        return wrapped->description() + ", milk";
    }
};

// Usage: Coffee* c = new MilkDecorator(new SimpleCoffee());
// c->cost() == 1.5, c->description() == "Simple coffee, milk"
```
