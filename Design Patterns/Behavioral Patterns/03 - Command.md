---
tags: [design, pattern, behavioral, command]
---

## Overview

Encapsulates a **request as an object**, thereby allowing you to parameterize clients with different requests, queue requests, log them, and support undoable operations.

## When to Use

- Undo/redo functionality
- Task queues
- Macro recording
- Callback abstraction

## Related Patterns

- [[09 - Memento]] (Memento can store state snapshots for Command's undo functionality)

## C++ Example

```cpp
class Command {
public:
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class Light {
public:
    void on() { /* turn on */ }
    void off() { /* turn off */ }
};

class LightOnCommand : public Command {
    Light* light;
public:
    LightOnCommand(Light* l) : light(l) {}
    void execute() override { light->on(); }
    void undo() override { light->off(); }
};

class RemoteControl {
    std::vector<Command*> history;
public:
    void pressButton(Command* cmd) {
        cmd->execute();
        history.push_back(cmd);
    }
    void pressUndo() {
        if (!history.empty()) {
            history.back()->undo();
            history.pop_back();
        }
    }
};
```
