---
tags:
  - design
  - pattern
  - creational
  - builder
---

## Overview

The Builder pattern separates the **construction** of a complex object from its **representation**, so the same construction process can create different representations. It is useful when an object requires many steps to create, or has many optional parameters.

## When to Use

- Constructing objects that would require a telescoping constructor (constructor with many parameters)
- When construction involves several steps

## Example (C++)

```cpp
class Pizza {
public:
    std::string dough, sauce, topping;
};

class PizzaBuilder {
protected:
    Pizza* pizza;
public:
    PizzaBuilder() { pizza = new Pizza(); }
    Pizza* getPizza() { return pizza; }

    virtual void buildDough() = 0;
    virtual void buildSauce() = 0;
    virtual void buildTopping() = 0;
};

class MargheritaBuilder : public PizzaBuilder {
public:
    void buildDough() override { pizza->dough = "thin crust"; }
    void buildSauce() override { pizza->sauce = "tomato"; }
    void buildTopping() override { pizza->topping = "mozzarella + basil"; }
};

class Cook {
    PizzaBuilder* builder;
public:
    void setBuilder(PizzaBuilder* b) { builder = b; }
    Pizza* makePizza() {
        builder->buildDough();
        builder->buildSauce();
        builder->buildTopping();
        return builder->getPizza();
    }
};
```
