---
tags: [design, pattern, behavioral, memento]
---

## Overview

Captures and externalizes an object's **internal state** so it can be **restored later**, without violating encapsulation.

## When to Use

- Undo mechanisms
- Saving game state
- Snapshots
- Transaction rollback

## Related Patterns

- [[03 - Command]] (Command can use Memento to store state for its undo operations)
