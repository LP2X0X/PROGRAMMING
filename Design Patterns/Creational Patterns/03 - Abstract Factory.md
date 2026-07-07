---
tags:
  - design
  - pattern
  - creational
  - abstract-factory
---

## Overview

The Abstract Factory pattern provides an interface for creating **families of related objects** without specifying their concrete classes. Think of it as a factory of factories.

## When to Use

- When your system needs to work with multiple families of related products
- Example: a UI toolkit that supports Windows and macOS widgets — buttons, checkboxes, menus — all need to match the same family

## Related Patterns

- [[02 - Factory Method]] — Abstract Factory uses Factory Methods internally

## Example (C++)

```cpp
// Abstract products
class Button { public: virtual void render() = 0; };
class Checkbox { public: virtual void render() = 0; };

// Concrete families
class WinButton : public Button { void render() override { /* Windows look */ } };
class WinCheckbox : public Checkbox { void render() override { /* Windows look */ } };

class MacButton : public Button { void render() override { /* macOS look */ } };
class MacCheckbox : public Checkbox { void render() override { /* macOS look */ } };

// Abstract factory
class GUIFactory {
public:
    virtual Button* createButton() = 0;
    virtual Checkbox* createCheckbox() = 0;
};

class WinFactory : public GUIFactory {
public:
    Button* createButton() override { return new WinButton(); }
    Checkbox* createCheckbox() override { return new WinCheckbox(); }
};
```
