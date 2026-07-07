---
tags:
  - design
  - pattern
  - creational
  - singleton
---

## Overview

The Singleton pattern ensures a class has **only one instance** and provides a global point of access to it. The class itself is responsible for keeping track of its sole instance.

## When to Use

- Logging
- Configuration
- Thread pools
- Caches
- Any resource where having multiple instances would cause conflicts or waste

## Example (C++)

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}  // private constructor
public:
    static Singleton* getInstance() {
        if (!instance)
            instance = new Singleton();
        return instance;
    }
};
```

## Caveats

```ad-warning
Singleton is often overused. It introduces global state, makes testing harder (hard to mock), and hides dependencies. Prefer dependency injection when possible.
```
