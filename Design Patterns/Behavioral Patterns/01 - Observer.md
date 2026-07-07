---
tags: [design, pattern, behavioral, observer]
---

## Overview

Defines a **one-to-many dependency** between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically.

## When to Use

- Event systems, pub/sub
- UI updates when model data changes (MVC)
- Any scenario where multiple objects need to react to state changes

## Related Patterns

- [[08 - Mediator]] (Mediator centralizes communication instead of direct observer subscriptions)

## C++ Example

```cpp
class Observer {
public:
    virtual void update(float temperature) = 0;
};

class WeatherStation {
    std::vector<Observer*> observers;
    float temperature;
public:
    void addObserver(Observer* o) { observers.push_back(o); }
    void setTemperature(float t) {
        temperature = t;
        for (auto* o : observers)
            o->update(temperature); // notify all
    }
};

class PhoneDisplay : public Observer {
public:
    void update(float temperature) override {
        // update phone screen with new temperature
    }
};
```
