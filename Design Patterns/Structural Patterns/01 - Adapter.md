---
tags:
  - design
  - pattern
  - structural
  - adapter
---

## Intent

Converts the interface of a class into **another interface that clients expect**. Lets classes work together that otherwise couldn't because of incompatible interfaces.

## Analogy

Think of it like a power plug adapter: the wall socket (existing interface) does not change, but the adapter lets your device (client) plug in.

## When to Use

- When you want to use an existing class but its interface does not match what you need.
- When integrating third-party or legacy code.

## Related Patterns

- [[03 - Facade]] (Facade simplifies, Adapter converts)

## Example (C++)

```cpp
// Existing interface your code expects
class MediaPlayer {
public:
    virtual void play(std::string filename) = 0;
};

// Incompatible third-party class
class VLCLibrary {
public:
    void playVLC(std::string filename) { /* plays .vlc files */ }
};

// Adapter bridges the gap
class VLCAdapter : public MediaPlayer {
    VLCLibrary* vlc;
public:
    VLCAdapter() { vlc = new VLCLibrary(); }
    void play(std::string filename) override {
        vlc->playVLC(filename);
    }
};
```
