---
tags:
  - design
  - pattern
  - creational
  - factory-method
---

## Overview

The Factory Method pattern defines an interface for creating an object, but lets **subclasses decide** which class to instantiate. The creator class has a method that returns a product, and subclasses override this method to produce different products.

## When to Use

- When a class cannot anticipate the type of objects it needs to create
- When you want subclasses to specify the objects they create

## Related Patterns

- [[03 - Abstract Factory]] — factory of factories builds on this concept

## Example (C++)

```cpp
class Transport {
public:
    virtual void deliver() = 0;
};

class Truck : public Transport {
public:
    void deliver() override { /* road delivery */ }
};

class Ship : public Transport {
public:
    void deliver() override { /* sea delivery */ }
};

class Logistics {
public:
    virtual Transport* createTransport() = 0;

    void planDelivery() {
        Transport* t = createTransport();
        t->deliver();
    }
};

class RoadLogistics : public Logistics {
public:
    Transport* createTransport() override {
        return new Truck();
    }
};
```
