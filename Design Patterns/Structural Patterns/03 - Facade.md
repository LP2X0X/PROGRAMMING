---
tags:
  - design
  - pattern
  - structural
  - facade
---

## Intent

Provides a **simplified interface** to a complex subsystem. It does not hide the subsystem -- it just provides a convenient entry point.

## When to Use

- When a subsystem has many classes and the client only needs a simple high-level interface.
- When you want to layer your system.

## Related Patterns

- [[01 - Adapter]] (Adapter converts interfaces, Facade simplifies them)

## Example (C++)

```cpp
// Complex subsystem classes
class CPU { public: void start() { } };
class Memory { public: void load() { } };
class HardDrive { public: void read() { } };

// Facade
class Computer {
    CPU cpu;
    Memory memory;
    HardDrive hd;
public:
    void startComputer() {
        cpu.start();
        memory.load();
        hd.read();
        // client doesn't need to know the boot sequence
    }
};
```
